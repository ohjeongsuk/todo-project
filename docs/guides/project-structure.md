# 프로젝트 구조 가이드

이 문서는 **`todo-frontend`(Next.js 16 App Router)**의 폴더 구조, 파일 조직 및 네이밍 컨벤션을 정의합니다.

> ⚠️ **스택 사실의 단일 출처는 `CLAUDE.md`(2·4장)다.** 이 가이드의 서술이 `CLAUDE.md`와 어긋나면 `CLAUDE.md`를 따른다.

## 이 프로젝트는 폴리레포다

`CLAUDE.md`, `PRD.md`, `ROADMAP.md` 등 프로젝트 문서는 `todo-project` 저장소의 `docs/` 아래 있고, `todo-frontend`는 **별도의 독립된 git 저장소**입니다. 즉 `todo-frontend` 안에서 `../PRD.md` 같은 상대 경로로 문서를 참조할 수 없습니다(그 경로에는 아무 파일도 없습니다). 문서를 가리킬 때는 "`todo-project` 저장소의 `docs/PRD.md`"처럼 저장소 이름을 함께 명시하십시오.

> ⚠️ 이전 버전의 이 가이드는 세 폴더(`todo-project`/`todo-backend`/`todo-frontend`)를 하나의 모노레포로 서술했습니다. 실제로는 세 개의 독립 git 저장소(폴리레포)이며, `todo-project`는 문서만 담습니다(`CLAUDE.md` 1장).

## `src/` 디렉토리를 사용하지 않는다

> `app/`, `components/`, `lib/`가 `todo-frontend/` 루트에 직접 위치합니다.
> (`components.json`의 alias가 `@/components`·`@/lib`이고 `tsconfig.json`의 `paths`가 `"@/*": ["./*"]`인 것과 일치)
> 다른 문서나 예시에서 `src/app/...` 경로를 보더라도, 이 프로젝트에서는 `src/`를 빼고 읽으십시오.

## 전체 구조 (폴리레포 3개 저장소)

```
todo-project/                 # 문서 전용 저장소
├── docs/
│   ├── PRD.md
│   ├── ROADMAP.md
│   └── guides/               # 개발 가이드 모음 (이 문서 포함)
└── CLAUDE.md                 # 개발 지침 메인 문서 (전체 정본)

todo-backend/                 # ⚙️ 별도 저장소 — Spring Boot (com.example.todoapp)

todo-frontend/                # 🚀 별도 저장소 — Next.js 16
├── app/                      # App Router (src/ 없음)
├── components/                # React 컴포넌트
├── lib/                       # 유틸리티 및 API 클라이언트
├── hooks/                     # React Query 훅
├── providers/                  # Context 프로바이더
├── types/                      # 공통 타입
├── public/                     # 정적 파일 (단, public/static/ 금지 — Amplify 예약 경로)
├── components.json             # shadcn/ui 설정
├── next.config.ts
├── tsconfig.json
└── package.json
```

## 세부 폴더 구조

### app/ — App Router 페이지

```
app/
├── layout.tsx                    # 루트 레이아웃 (Providers)
├── page.tsx                      # 진입 — 토큰 유무로 리다이렉트
├── globals.css                   # Tailwind 4 CSS-first 설정 + 디자인 토큰
├── (auth)/                       # 비인증 라우트 그룹
│   ├── login/page.tsx
│   └── signup/page.tsx
├── (main)/                       # 인증 필요 라우트 그룹
│   ├── layout.tsx                # 인증 가드(클라이언트) + 공통 헤더
│   └── todos/
│       ├── page.tsx              # Todo 목록 (페이지네이션)
│       ├── new/page.tsx          # Todo 작성
│       └── [id]/page.tsx         # Todo 상세/편집
└── oauth2/callback/page.tsx      # OAuth2 콜백 (토큰 수신)
```

- `page.tsx`: 해당 경로의 메인 페이지
- `layout.tsx`: 레이아웃 컴포넌트(자식 페이지 감쌈)
- `(그룹명)/`: URL에 영향을 주지 않는 라우트 그룹 — 인증/비인증 레이아웃 분리에 사용
- `middleware.ts`/`proxy.ts`는 두지 않는다(`nextjs-app-router.md` 참조)

### components/ — 컴포넌트 조직

```
components/
├── ui/                     # 기본 UI 컴포넌트 (shadcn/ui, 자동 생성)
│   ├── button.tsx
│   ├── card.tsx
│   └── ...
├── common/                 # 전역 공통 컴포넌트
│   ├── Pagination.tsx     # 재사용 페이지네이션
│   ├── EmptyState.tsx
│   ├── ErrorState.tsx
│   └── Skeleton.tsx
├── auth/                   # 인증 관련
│   ├── LoginForm.tsx
│   └── SignupForm.tsx
├── todo/                   # Todo 도메인
│   ├── TodoList.tsx
│   ├── TodoItem.tsx
│   └── TodoForm.tsx
└── editor/                 # Tiptap 에디터
    └── TiptapEditor.tsx
```

`providers/`는 `components/` 안이 아니라 **루트의 별도 디렉토리**입니다.

### lib/ — 유틸리티 및 API 클라이언트

```
lib/
├── api/
│   ├── client.ts          # apiClient — 토큰 주입, ApiResponse 언래핑, 401 처리
│   ├── auth.ts
│   └── todo.ts
├── auth/token.ts           # 토큰 저장/조회/삭제
├── validation.ts           # 입력값 검증 규칙 (폼 라이브러리 없이 직접 관리)
├── errorMessages.ts        # error.code → 화면 문구 매핑
├── sanitize.ts             # DOMPurify 래퍼
└── utils.ts                # cn() 등
```

> **모든 API 호출은 `lib/api/`를 경유**합니다. 컴포넌트에서 `fetch`를 직접 호출하지 않습니다. 서버 상태는 `hooks/`의 React Query 훅으로 관리합니다.
> 이 프로젝트는 폼 라이브러리(`react-hook-form`, `zod`)를 쓰지 않으므로 `lib/schemas/`가 없습니다. 대신 `lib/validation.ts`에 검증 함수를 모읍니다(`forms.md` 참조).

## 파일 네이밍 컨벤션

`CLAUDE.md` 4장에 명시된 규칙입니다.

- 컴포넌트 파일명: `PascalCase.tsx`
- 유틸/훅 파일명: `camelCase.ts`
- 폴더명: 소문자 또는 kebab-case

```tsx
// ✅ 올바른 컴포넌트 네이밍
export function LoginForm() {} // PascalCase
export function useTodos() {}  // camelCase 훅

// ❌ 잘못된 네이밍
export function login_form() {} // snake_case 금지
```

## 경로 별칭 (Path Aliases)

`tsconfig.json`의 `paths`와 `components.json`의 alias는 모두 `src/`를 거치지 않는 `@/*` → `./*` 구조입니다.

```typescript
// ✅ 경로 별칭 사용
import { Button } from '@/components/ui/button'
import { cn } from '@/lib/utils'

// ❌ 상대 경로 사용
import { Button } from '../../../components/ui/button'
```

## 코드 조직 베스트 프랙티스

### 의존성 import 순서

```typescript
// 1. 외부 라이브러리
import { useState } from 'react'

// 2. 내부 라이브러리 (@/ 경로)
import { Button } from '@/components/ui/button'
import { cn } from '@/lib/utils'

// 3. 상대 경로
import './component.css'
```

### 파일 크기

단일 파일은 300줄 이하를 권장합니다. 초과하면 분할을 고려합니다.

## 금지사항

```bash
# 깊은 중첩 구조 (4단계 이상)
components/pages/auth/forms/login/LoginForm.tsx

# 의미 없는 폴더명
components/misc/
```

```typescript
// 깊은 상대 경로
import { utils } from '../../../../../lib/utils'
```

## 체크리스트

- [ ] `src/` 없이 `app/`, `components/`, `lib/`가 루트 직하에 있는지
- [ ] 적절한 카테고리 폴더에 배치했는지
- [ ] 경로 별칭(`@/*`) 사용, 상대 경로 금지
- [ ] 컴포넌트는 PascalCase.tsx, 유틸/훅은 camelCase.ts
- [ ] `public/static/`을 만들지 않았는지(Amplify 예약 경로)

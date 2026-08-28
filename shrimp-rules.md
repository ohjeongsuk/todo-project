# Development Guidelines

Todo List 서비스(모노레포: Spring Boot 4 백엔드 + Next.js 16 프론트엔드)에서 **AI Agent가 코드를 수정할 때 지켜야 하는 규칙**이다.
기능 설명은 이 문서의 역할이 아니다. "무엇을 만드는가"는 `docs/PRD.md`, "언제 만드는가"는 `docs/ROADMAP.md`를 읽는다.

---

## 1. 문서 우선순위

충돌 시 위쪽이 이긴다.

| 순위 | 문서 | 역할 |
|---|---|---|
| 1 | `D:\claude\todo-project\CLAUDE.md` | 절대 규칙 11개, 컨벤션 |
| 2 | `docs/PRD.md` | 요구사항(F-xx / NF-xx), 화면 목록 |
| 3 | `docs/ROADMAP.md` | Phase 순서, 의존성, 완료 판정 |
| 4 | `docs/DEV_TOOLS.md` | 도구 명령어 |
| 5 | `docs/guides/*.md` | 프론트 구현 참고 — **오류가 확인된 문서** |

### guides 신뢰 금지 항목

`docs/guides/`는 아래 오류가 확인됐다. 그대로 따르지 않는다.

- `forms-react-hook-form.md` 40번 줄: react-hook-form·zod·@hookform/resolvers가 "이미 설치되어 있습니다"라고 하나 **셋 다 미설치**. 설치 여부는 항상 `package.json`을 직접 읽어 확인한다.
- `styling-guide.md` 6번 줄: "PRD 1.3의 설치 상태 열"을 참조하나 **PRD에 그런 절도 열도 없다**. `docs/ROADMAP.md` 3.2절을 대신 본다.
- `styling-guide.md` 17·20번 줄: "M5"라는 표기는 이 프로젝트의 단계 체계가 아니다. **"Phase 6"으로 읽는다.**
- `project-structure.md` 55번 줄: OAuth 콜백을 `app/oauth2/callback/page.tsx`로 두나, PRD 6장이 정한 경로는 **`/auth/callback`**이다. PRD를 따른다.
- `project-structure.md`: `todos/[id]/page.tsx` 상세 페이지를 두나, PRD 6장은 **"별도 상세 페이지 없음, 다이얼로그로 처리"**다. 만들지 않는다.
- `project-structure.md`: `docs/API_SPEC.md`를 참조하나 실제 파일명은 **`docs/API.md`**다.

---

## 2. 저장소 지형

```
todo-project/
├── CLAUDE.md, shrimp-rules.md(이 문서)
├── docs/            PRD.md ROADMAP.md DEV_TOOLS.md guides/
├── todo-backend/    Spring Boot 4.1.1 / JDK 21 / Maven
└── todo-frontend/   Next.js 16.3.3 / React 19.2.8 / Tailwind 4
```

### 읽기·수정 금지 경로

| 경로 | 이유 |
|---|---|
| `.metadata/` | Eclipse 워크스페이스 메타데이터. 소스가 아니다. 커밋 대상도 아니다 |
| `todo-frontend/.next/` | 빌드 산출물. 단 `.next/types/*.d.ts`는 **읽기 전용 참조용**으로 유효하다 |
| `todo-backend/target/` | 빌드 산출물 |
| `todo-frontend/AGENTS.md` | `next dev`가 자동 재생성한다. 삭제해도 다시 생기므로 **지우려 하지 않는다** |
| `todo-frontend/components/ui/` | shadcn CLI 생성물. 직접 편집하지 않고 CLI로 재생성한다 |
| `shrimp_data/` | Shrimp Task Manager 데이터 |

---

## 3. 백엔드 규칙 (`todo-backend/`)

### 3.1 Spring Boot 4 스타터명 — 학습된 Boot 3 이름을 쓰지 않는다

`pom.xml`에 실제로 있는 이름을 그대로 쓴다.

| 쓴다 (Boot 4) | 쓰지 않는다 (Boot 3) |
|---|---|
| `spring-boot-starter-webmvc` | ~~`spring-boot-starter-web`~~ |
| `spring-boot-starter-security-oauth2-client` | ~~`spring-boot-starter-oauth2-client`~~ |
| `spring-boot-starter-webmvc-test` | ~~`spring-boot-starter-test`~~ |
| `spring-boot-starter-data-jpa-test` | ~~`spring-boot-starter-test`~~ |
| `spring-boot-starter-security-test` | ~~`spring-boot-starter-test`~~ |

테스트 스타터가 기능별로 쪼개져 있다. 새 테스트를 추가할 때 필요한 `-test` 스타터가 `pom.xml`에 있는지 먼저 확인한다.

### 3.2 Spotless가 빌드를 실패시킨다

- `spotless:check`가 **`validate` 단계에 묶여 있다.** 포맷이 틀리면 컴파일 전에 빌드가 죽는다.
- 코드를 쓴 뒤 반드시 `./mvnw spotless:apply`를 실행한다.
- 포맷: **AOSP 스타일, 들여쓰기 4칸, 100자.** 임의로 다른 포맷을 쓰지 않는다.

### 3.3 코드 규칙

- 계층은 `controller → service → repository`. 컨트롤러가 리포지토리를 직접 호출하지 않는다.
- 패키지는 기능별로 나눈다: `auth`, `user`, `todo`, `global`.
- DTO는 **record**, 엔티티는 **class**.
- 엔티티에 `@Setter`를 열지 않는다. 상태 변경은 `complete()`, `updateContent()` 같은 의미 있는 메서드로 표현한다.
- 컨트롤러가 엔티티를 반환하지 않는다. 항상 DTO로 변환한다.
- 생성자 주입만 쓴다. `@Autowired` 필드 주입 금지.
- 조회 메서드에 `@Transactional(readOnly = true)`를 붙인다.
- `Optional`은 반환 타입으로만 쓴다. 필드·파라미터로 쓰지 않는다.
- 예외 응답은 `GlobalExceptionHandler` **한 곳에서만** 만든다. 컨트롤러에서 try-catch로 응답을 조립하지 않는다.
- 소유권 검증은 **쿼리 단계에서** `user_id`로 강제한다. 조회 후 자바 코드로 비교하지 않는다.
- 소유권 위반은 **404**로 응답한다. 403이 아니다.
- 삭제는 전부 Soft Delete(`deleted_at`). `DELETE FROM` 금지.

---

## 4. 프론트엔드 규칙 (`todo-frontend/`)

### 4.1 Next.js 16 — 학습된 v15 지식과 다른 지점

| 항목 | 이 프로젝트의 사실 |
|---|---|
| 레이아웃 Props | `app/layout.tsx`는 **`LayoutProps<"/">`**를 쓴다. Next 16이 `.next/types/routes.d.ts`에 자동 생성하는 전역 타입이다. `{ children }: { children: React.ReactNode }`로 되돌리지 않는다 |
| 미들웨어 | v16에서 `middleware.ts` → **`proxy.ts`**, 함수명도 `middleware` → **`proxy`**. Node.js 런타임이 기본값 |
| `params` / `searchParams` | **Promise**다. `await` 해서 쓴다 |
| `useSearchParams` | 프로덕션 빌드 시 **Suspense 경계 필수**. 없으면 "Missing Suspense boundary" 빌드 실패 |

### 4.2 디렉터리

- **`src/` 디렉터리를 만들지 않는다.** `app/`, `components/`, `lib/`, `hooks/`, `providers/`, `types/`가 `todo-frontend/` 루트 직속이다.
- import alias는 `@/*` → `./*`.
- 파일명: 컴포넌트는 `PascalCase.tsx`, 훅·유틸은 `camelCase.ts`.

### 4.3 TypeScript

- `strict: true`에 더해 **`noUncheckedIndexedAccess: true`**가 켜져 있다. 배열/객체 인덱싱 결과가 `T | undefined`이므로 그대로 쓰면 타입 에러가 난다. 좁히기(narrowing) 없이 인덱싱 결과를 사용하지 않는다.
- `noUnusedLocals`, `noUnusedParameters`가 켜져 있다. 미사용 변수는 빌드 실패 요인이다.
- `any` 금지 (ESLint `error`로 강제됨). 불가피하면 `unknown` + 타입가드.
- 서버 응답 타입은 `types/`에 **한 번만** 정의한다. 컴포넌트 안에서 인라인으로 다시 선언하지 않는다.
- 미사용 인자를 의도적으로 남길 때만 `_` 접두사를 쓴다.

### 4.4 Tailwind 4 / shadcn

- **CSS-first다. `tailwind.config.js` / `tailwind.config.ts`를 만들지 않는다.** 설정은 전부 `app/globals.css`에 있다.
- 토큰은 `@theme inline` 블록에 정의한다. v3의 `theme.extend` 문법을 쓰지 않는다.
- 다크모드는 `@custom-variant dark (&:is(.dark *))` — **`.dark` 클래스 기반**이다. `next-themes`를 도입하면 `attribute="class"`로 설정한다.
- shadcn 설정: style `radix-nova`, baseColor `neutral`, rsc `true`, icon `lucide`.
- Radix는 개별 `@radix-ui/*`가 아니라 **통합 `radix-ui` 패키지**가 설치돼 있다. 개별 패키지를 새로 추가하지 않는다.

### 4.5 컴포넌트 경계

- 기본은 **서버 컴포넌트**다. `'use client'`는 이벤트 핸들러·브라우저 API·상태가 실제로 필요한 컴포넌트에만 붙인다.
- 데이터 패칭은 React Query 훅으로 통일한다. 컴포넌트 안에서 `fetch`를 직접 호출하지 않는다.

### 4.6 포맷

Prettier 설정을 임의로 바꾸지 않는다. 코드를 쓸 때 이 값에 맞춘다.

| 항목 | 값 |
|---|---|
| 따옴표 | **쌍따옴표** (`singleQuote: false`) |
| 세미콜론 | 있음 |
| 줄 길이 | 100 |
| 들여쓰기 | 2칸 |
| 개행 | **LF** |
| trailingComma | `all` |

- `.gitattributes`가 `* text=auto eol=lf`다. **CRLF로 저장하면 `prettier --check`가 계속 실패한다.** Windows 환경이므로 실제로 자주 발생한다.
- `cn`, `cva`, `clsx`, `twMerge` 안의 클래스 문자열은 `prettier-plugin-tailwindcss`가 정렬한다. 수동으로 순서를 맞추지 않는다.

### 4.7 커밋 훅

`.husky/pre-commit` → `lint-staged.config.mjs`가 스테이징된 파일에 아래를 실행한다.

| 대상 | 실행 내용 |
|---|---|
| `*.{ts,tsx}` | `tsc --noEmit` → `eslint --fix --max-warnings=0` → `prettier --write` |
| `*.{js,jsx,mjs,cjs}` | `eslint --fix --max-warnings=0` → `prettier --write` |
| `*.{css,json,md,mdx,yml,yaml}` | `prettier --write` |

- **`--max-warnings=0`이므로 경고 하나만 있어도 커밋이 막힌다.** 미사용 변수(`@typescript-eslint/no-unused-vars`)와 `console.log`(`no-console`)가 경고라서 실제로 자주 걸린다. `console.warn`/`console.error`만 허용된다.
- `lint-staged.config.mjs`의 `tsc` 항목은 `() => "tsc --noEmit"` 형태로 감싸져 있다. **이 화살표 함수를 풀지 않는다.** 풀면 `tsc`에 파일 경로가 주입되어 `tsconfig.json`이 통째로 무시되고 대량 오탐이 발생한다.
- `.husky/**`는 `eol=lf`여야 한다. CRLF면 훅이 `sh`에서 실행되지 않는다.

커밋 메시지는 `commitlint.config.mjs`가 검사한다: Conventional Commits 타입 11종, 제목 100자 이내, **제목 끝에 마침표 금지**, 한글 제목 허용(`subject-case` 비활성).

---

## 5. 다중 파일 연동 규칙

**왼쪽을 수정하면 오른쪽도 같은 작업 안에서 수정한다.**

| 수정 대상 | 함께 수정할 파일 | 안 하면 |
|---|---|---|
| 백엔드 패키지 이동 (`com.example` → `com.example.todoapp`) | `todo-backend/pom.xml`의 Spotless `<importOrder>` 안 `com.example` 항목 | import 정렬이 어긋나 `validate`에서 빌드 실패 |
| `app/**/page.tsx` 신규 생성 | 같은 폴더의 `loading.tsx`, `error.tsx` | 로딩·에러 화면 없이 "완료" 판정됨 (ROADMAP 원칙 6) |
| 서버 응답 형태 변경 | `types/`의 해당 타입 정의 1곳 | 컴포넌트마다 타입이 갈라짐 (NF-17 위반) |
| 요구사항 신설·변경 | `docs/PRD.md`(F-xx 부여) + `docs/ROADMAP.md` **3장 요약표와 4장 매핑표 둘 다** | 두 표가 어긋난다. 이 저장소에서 실제로 발생했던 사고다 |
| `docs/CHECKLIST.md` 항목 번호 변경 | `docs/ROADMAP.md` 3장 "완료 판정" 칼럼 | 완료 판정이 빈 참조가 된다 |
| 새 라이브러리 추가 | 먼저 **사용자 승인**을 받는다 (CLAUDE.md 규칙 11). 승인 대상 목록은 `docs/ROADMAP.md` 3.2절 | 규칙 11 위반 |
| `application.properties` 수정 | 비밀값은 환경변수 또는 `application-local.properties`로 분리 | 비밀 유출 (NF-05 위반) |

---

## 6. AI 의사결정 기준

모호하면 아래 순서로 판단한다.

| 상황 | 판단 |
|---|---|
| 문서끼리 충돌한다 | 1장 우선순위표를 따른다. guides가 최하위다 |
| 라이브러리 API가 기억과 다른 것 같다 | **기억을 버린다.** Context7 또는 공식 문서를 확인한다. Boot 4 / Security 7 / Next 16 / Tailwind 4는 전부 메이저 변경이라 학습 지식이 틀릴 가능성이 높다 |
| 이 라이브러리가 설치돼 있나? | `package.json` 또는 `pom.xml`을 **직접 읽는다.** 가이드 문서의 서술을 믿지 않는다 |
| 데이터를 지워야 한다 | Soft Delete(`deleted_at`). 물리 삭제는 어떤 경우에도 하지 않는다 |
| 남의 리소스에 접근했다 | **404**. 403이 아니다 (존재 여부 비노출) |
| 요구사항이 모호하다 | 추측해서 만들지 않는다. 질문한다 |
| 기존 파일을 고치는 중 다른 문제가 보인다 | 관련 없는 코드를 함께 리팩터링하지 않는다 |
| 화면 경로를 새로 만들어야 할 것 같다 | PRD 6장 화면 목록에 없으면 만들지 않는다 |
| Phase 순서를 건너뛰고 싶다 | `docs/ROADMAP.md` 2장 의존성 규칙을 확인한다. 인증(Phase 0~3) 전에 Todo에 손대지 않는다 |

---

## 7. 금지 사항

- ❌ `tailwind.config.js` / `tailwind.config.ts` 생성 (CSS-first다)
- ❌ `src/` 디렉터리 생성
- ❌ `todo-frontend/AGENTS.md` 삭제 시도 (자동 재생성됨)
- ❌ Dockerfile, docker-compose.yml, Testcontainers 사용 — **명시적으로 배제됨**
- ❌ Amazon S3 / 파일 업로드 기능 추가 — 이번 범위 밖
- ❌ NextAuth / Auth.js 도입 — 인증은 백엔드 주도 OAuth2다
- ❌ 엔티티에 `@Setter` 추가
- ❌ 컨트롤러에서 엔티티 직접 반환
- ❌ 컨트롤러에서 try-catch로 에러 응답 조립
- ❌ TypeScript `any`
- ❌ 컴포넌트 안에서 `fetch` 직접 호출
- ❌ 비밀·키 하드코딩
- ❌ 승인 없는 라이브러리 추가
- ❌ 물리 삭제(`DELETE FROM`)
- ❌ CORS 와일드카드(`*`) — `allowCredentials(true)`와 함께 쓸 수 없다
- ❌ Refresh Token을 localStorage나 응답 본문에 저장
- ❌ 영문 주석 (주석은 한글, 식별자는 영문)

---

## 8. 검증 명령

작업을 끝냈다고 보고하기 전에 실제로 실행한다.

```bash
# 프론트엔드 (todo-frontend/)
npm run check          # type-check + lint + format:check 일괄
npm run format         # 포맷 자동 수정

# 백엔드 (todo-backend/)
./mvnw spotless:apply  # 포맷 자동 수정 (빌드 전 필수)
./mvnw verify          # 빌드 + 테스트
```

- 백엔드는 `spotless:apply`를 먼저 하지 않으면 `validate`에서 실패한다.
- JDK 21이 아니면 maven-enforcer가 즉시 실패시킨다. `JAVA_HOME`을 확인한다.

---

## 9. ⏳ 전환 중 — 조건부 규칙

**아래는 현재 미해소 상태에만 유효하다. 해소되면 해당 줄을 이 문서에서 삭제한다.**

| 현재 상태 | 작업 시 주의 | 해소 시점 |
|---|---|---|
| 백엔드 패키지가 `com.example` (`TodoBackendApplication.java`) | 새 클래스는 이동 후 구조(`com.example.todoapp.*`)를 전제로 만들지 말고, 이동 작업과 함께 처리한다 | ROADMAP Phase 0 |
| `pom.xml`에 **Jsoup·SpringDoc 없음** | 서버 sanitize(F-26)·Swagger(F-35~F-37) 코드를 쓰기 전에 승인 및 의존성 추가가 선행돼야 한다 | ROADMAP Phase 0 |
| TanStack Query·Framer Motion·Tiptap·isomorphic-dompurify·next-themes **미설치** | 이 라이브러리를 `import`하는 코드를 쓰지 않는다. 설치는 승인 후 (`docs/ROADMAP.md` 3.2절) | Phase 6·8 |
| `docs/SCHEMA.md`·`API.md`·`DESIGN.md` 미작성 | 각각 Phase 1·2·6의 산출물이다. 해당 Phase 작업 시 함께 만든다 | Phase 1·2·6 |
| `PROMPTS.md` **미작성** | ROADMAP 머리말이 참조하나 존재하지 않는다. 이 파일을 찾지 말 것 | 작성 시 |

### 최근 해소된 항목 (참고용, 더 이상 주의 불필요)

- DB는 `todolist_db` **별도 데이터베이스**로 이미 분리됨 (`application.properties` 확인).
- `todo-frontend/.git` 중첩 저장소 **없음**.
- 루트 `.gitignore` **존재함**.
- `docs/CHECKLIST.md` **작성 완료**(409줄). ROADMAP 3장 완료 판정 칼럼이 참조 가능.

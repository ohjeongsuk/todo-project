# Next.js 16 App Router 개발 지침

이 문서는 Next.js 16(App Router)으로 이 프로젝트를 개발할 때 따라야 할 규칙을 정리합니다.

> ⚠️ **스택 사실의 단일 출처는 `CLAUDE.md`(2·4장)다.** 이 가이드의 서술이 `CLAUDE.md`와 어긋나면 `CLAUDE.md`를 따른다.
> 버전은 `todo-frontend/package.json`을 실제 기준으로 삼는다(`next` 16.x). 재조사하지 않는다.

## 이 프로젝트의 전제: 클라이언트 컴포넌트 우선

일반적인 Next.js App Router 가이드는 "Server Components를 기본값으로, 상호작용이 필요한 부분만 Client Component로 분리"를 권장합니다. **이 프로젝트는 그 반대를 기본값으로 삼습니다.**

이유는 인증 구조에 있습니다. Access Token은 브라우저 `localStorage`(`todo_access_token`)에 저장되고, 서버 컴포넌트는 브라우저 저장소에 접근할 수 없습니다. 즉 서버 컴포넌트에서 `fetch('/api/v1/todos')`를 호출해도 인증 헤더를 실을 방법이 없어, 사용자별 데이터(할 일 목록, 프로필 등)는 애초에 서버에서 가져올 수 없습니다.

```typescript
// ❌ 이 프로젝트에서는 동작하지 않는다 — 서버 컴포넌트는 localStorage를 못 읽는다
export default async function TodosPage() {
  const token = localStorage.getItem('todo_access_token') // ReferenceError: localStorage is not defined
  const todos = await fetch('/api/v1/todos', {
    headers: { Authorization: `Bearer ${token}` },
  })
  // ...
}

// ✅ 이 프로젝트의 방식 — 클라이언트 컴포넌트 + React Query 훅
'use client'

export default function TodosPage() {
  const { data: todos } = useTodos() // 내부에서 apiClient가 localStorage 토큰을 첨부
  // ...
}
```

그래서 이 프로젝트의 페이지(`page.tsx`)는 대부분 `'use client'`를 갖습니다. 서버 컴포넌트로 남겨도 좋은 것은 인증과 무관한 순수 레이아웃 껍데기 정도입니다(루트 `layout.tsx` 등).

## App Router 기본 구조

```
app/
├── layout.tsx          # 루트 레이아웃 (서버 컴포넌트)
├── page.tsx            # 진입 — 토큰 유무로 리다이렉트
├── globals.css
├── (auth)/              # 비인증 라우트 그룹
│   ├── login/page.tsx
│   └── signup/page.tsx
├── (main)/              # 인증 필요 라우트 그룹
│   ├── layout.tsx       # 클라이언트 레이아웃, 라우트 보호
│   └── todos/
│       ├── page.tsx
│       ├── new/page.tsx
│       └── [id]/page.tsx
└── oauth2/callback/page.tsx
```

`pages/` 디렉토리 방식(Pages Router)은 쓰지 않습니다. `getServerSideProps`/`getStaticProps`도 이 프로젝트에는 없습니다.

## async 요청 API

`params`, `searchParams`, `cookies()`, `headers()`는 모두 Promise로 제공됩니다. 동기 접근은 지원하지 않습니다.

```typescript
export default async function Page({
  params,
  searchParams,
}: {
  params: Promise<{ id: string }>
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>
}) {
  const { id } = await params
  const query = await searchParams
  // ...
}
```

이 프로젝트에서 `params`를 쓰는 대표적인 곳은 `app/(main)/todos/[id]/page.tsx`(할 일 상세)입니다. 이 페이지는 클라이언트 컴포넌트이므로 서버 컴포넌트 시그니처(`async function` + `await params`)를 쓰지 않고, `useParams()`(next/navigation)로 클라이언트에서 라우트 파라미터를 읽습니다.

```typescript
'use client'

import { useParams } from 'next/navigation'

export default function TodoDetailPage() {
  const { id } = useParams<{ id: string }>()
  // ...
}
```

## `useSearchParams`와 Suspense

`useSearchParams()`를 쓰는 클라이언트 컴포넌트는 반드시 `<Suspense>`로 감싸야 합니다. 감싸지 않으면 `npm run build`가 실패합니다. 이 프로젝트에서는 `/oauth2/callback`(토큰 파라미터를 읽음)과 `/todos` 목록 페이지(검색·필터·페이지를 URL 쿼리로 관리)가 여기에 해당합니다.

```typescript
// app/oauth2/callback/page.tsx
import { Suspense } from 'react'

export default function OAuthCallbackPage() {
  return (
    <Suspense fallback={<CallbackSkeleton />}>
      <OAuthCallbackContent />
    </Suspense>
  )
}

function OAuthCallbackContent() {
  'use client'
  const searchParams = useSearchParams()
  // ...
}
```

## Streaming과 Suspense

느린 데이터를 기다리는 동안 나머지 UI를 먼저 보여줄 때 사용합니다. React Query를 쓰는 클라이언트 컴포넌트에서는 주로 `isLoading` 분기로 스켈레톤을 보여주는 방식을 더 많이 쓰지만, `<Suspense>` 자체는 `useSearchParams` 경계용으로 반드시 필요합니다.

## 이 프로젝트가 쓰지 않는 기능 (금지)

일반적인 Next.js 16 프로젝트에는 있지만, 이 프로젝트의 아키텍처(Spring Boot 백엔드 + localStorage 토큰)와 맞지 않아 쓰지 않는 기능입니다. 코드에서 아래 항목을 보면 스택 원칙 위반으로 간주합니다.

| 기능 | 쓰지 않는 이유 |
|---|---|
| Server Actions (`'use server'`) | 모든 쓰기 작업은 Spring Boot REST API(`POST /api/v1/...`)를 React Query mutation으로 호출한다. Next.js 서버가 DB나 인증 로직을 직접 갖지 않는다 |
| Route Handlers (`app/api/**/route.ts`) | 백엔드 API는 Spring Boot(`localhost:8080`)에 있다. Next.js에 자체 API 레이어를 두지 않는다 |
| `middleware.ts` / `proxy.ts` | 라우트 보호는 `(main)` 클라이언트 레이아웃에서 처리한다. `localStorage`는 미들웨어/프록시 레이어(엣지 런타임)에서 읽을 수 없기 때문이다 |
| Parallel Routes (`@slot`) / Intercepting Routes (`(.)`) | 이 프로젝트의 화면 구성에는 동시 렌더링 슬롯이나 모달 인터셉트가 필요한 화면이 없다 |
| ISR (`revalidate`, `fetch`의 `next.revalidate`) | Todo 데이터는 사용자별 인증 데이터라 캐싱 대상이 아니다. 정적 재생성이 의미 있는 페이지가 없다 |
| Cache Components (`cacheComponents`, `'use cache'`) | 위와 같은 이유로 캐싱 대상 데이터가 없어 켤 필요가 없다. `next.config.ts`에도 설정하지 않는다 |
| `notFound()` (next/navigation) | 타인 소유 Todo 접근 시 Next.js의 기본 404 처리 대신, 전용 화면("찾을 수 없습니다" + 목록으로 가기 버튼)을 직접 렌더링한다(`TODO-14`) |
| `unauthorized()` / `forbidden()` | 인증 실패는 401 응답을 받은 `apiClient`가 처리하고 `/login`으로 리다이렉트한다. Next.js의 401/403 페이지 API는 쓰지 않는다 |

## Turbopack / 패키지 import 최적화

기본 `next dev`, `next build`가 Turbopack을 사용합니다. 별도 설정이 필요하면 `next.config.ts` 최상위에 `turbopack` 옵션을 둡니다(`experimental.turbo` 아님).

```typescript
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  experimental: {
    optimizePackageImports: ['lucide-react', 'date-fns'],
  },
}

export default nextConfig
```

## 코드 품질 체크리스트

```bash
npm run type-check
npm run lint
npm run format:check
npm run build
```

`npm run build`가 성공해야 `useSearchParams` Suspense 경계 누락 같은 문제를 잡아낼 수 있습니다. 개발 서버(`next dev`)만으로는 이 오류가 드러나지 않습니다.

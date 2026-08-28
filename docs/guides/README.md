# 개발 가이드 모음

이 폴더는 `todo-frontend` 개발 시 참고할 상세 가이드를 모아둡니다.

> ⚠️ **이 가이드들은 참고 자료다. `CLAUDE.md`와 서술이 충돌하면 항상 `CLAUDE.md`가 우선한다.**
> 각 문서 상단에도 동일한 경고가 반복되어 있습니다.

## 문서 목록

| 문서 | 다루는 내용 |
|---|---|
| [`nextjs-app-router.md`](./nextjs-app-router.md) | App Router 구조, 클라이언트 컴포넌트 우선 전략, 이 프로젝트가 쓰지 않는 Next.js 16 기능 |
| [`component-patterns.md`](./component-patterns.md) | 컴포넌트 설계·재사용 패턴, Props 타입, 메모이제이션 |
| [`styling-guide.md`](./styling-guide.md) | Tailwind CSS 4 사용법, 미디어쿼리 기반 다크모드, shadcn/ui 활용 |
| [`project-structure.md`](./project-structure.md) | `todo-frontend`의 폴더 구조, 네이밍 컨벤션, 경로 별칭 |
| [`forms.md`](./forms.md) | 폼 라이브러리 없이 `useState` + 수동 검증으로 폼을 처리하는 방법 |

## 재작성 이력

**2026-08-28** — 5개 문서 전부 이 프로젝트 기준으로 전면 재작성했습니다. 원본은 다른 프로젝트(Server Actions 기반, `react-hook-form`+`zod` 폼, `src/` 디렉토리, `next-themes` 다크모드 토글을 전제)에서 넘어온 범용 템플릿 문서였고, 이 프로젝트의 실제 스택(Spring Boot 백엔드, `localStorage` 토큰, 폼 라이브러리 미사용, 미디어쿼리 다크모드, `src/` 미사용)과 맞지 않았습니다.

- `nextjs-16.md` → `nextjs-app-router.md`로 개명 및 재작성 (Server Components 우선 → 클라이언트 컴포넌트 우선으로 전환, Server Actions·Route Handlers·미들웨어 등 미사용 기능을 금지 표로 명시)
- `forms-react-hook-form.md` → `forms.md`로 개명 및 재작성 (폼 라이브러리 없는 수동 검증 패턴, 비밀번호 UTF-8 72바이트 제약 추가)
- `component-patterns.md`, `styling-guide.md`, `project-structure.md`는 파일명을 유지한 채 스택 관련 서술만 이 프로젝트 기준으로 교정

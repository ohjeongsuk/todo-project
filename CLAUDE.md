# CLAUDE.md

이 파일은 Claude Code가 이 저장소에서 작업할 때 **항상 지켜야 하는 규칙**이다.
개별 작업 지시(프롬프트)와 충돌할 경우, 이 파일이 우선한다.

---

## 1. 프로젝트 개요

로컬 개발 후 AWS로 배포하는 풀스택 Todo List 서비스.

```
todo-project/
├── CLAUDE.md
├── docs/
│   ├── API.md         # 엔드포인트 계약
│   ├── SCHEMA.md      # DB 스키마 / 엔티티
│   ├── DESIGN.md      # UI 디자인 토큰 및 규칙
│   └── CHECKLIST.md   # 최종 검증 체크리스트
├── todo-backend/      # Spring Boot
└── todo-frontend/     # Next.js
```

**저장소는 하나(모노레포)이며, 백엔드와 프론트엔드를 위 두 폴더로 분리한다.**

---

## 2. 기술 스택 (버전 고정)

### 백엔드 (`todo-backend/`)
| 항목 | 버전/선택 |
|---|---|
| Java | JDK 21 |
| 프레임워크 | Spring Boot 4.x |
| 빌드 | Maven (`mvnw` 래퍼 사용) |
| 보안 | Spring Security (Boot 4에 대응하는 Spring Security 버전) |
| ORM | Spring Data JPA / Hibernate |
| DB | PostgreSQL |
| API 문서 | SpringDoc OpenAPI (Swagger UI) |
| 검증 | Jakarta Bean Validation |
| HTML Sanitize | Jsoup |

### 프론트엔드 (`todo-frontend/`)
| 항목 | 버전/선택 |
|---|---|
| 프레임워크 | Next.js 16 (App Router) |
| 라이브러리 | React 19 |
| 언어 | TypeScript (strict) |
| 스타일 | Tailwind CSS 4 |
| 컴포넌트 | shadcn/ui |
| 아이콘 | lucide-react |
| 서버 상태 | TanStack Query (React Query) v5 |
| 애니메이션 | Framer Motion |
| 에디터 | Tiptap |
| Sanitize | DOMPurify (isomorphic-dompurify) |

### 인프라
- 로컬: 직접 설치한 PostgreSQL
- 배포(후순위): 프론트 = AWS Amplify, 백엔드 = EC2, DB = RDS
- **Docker는 사용하지 않는다.** Dockerfile, docker-compose.yml, Testcontainers 모두 금지.
- **Amazon S3는 이번 범위에서 사용하지 않는다.** (파일 업로드 기능 없음)

---

## 3. 절대 규칙 (위반 시 되돌릴 것)

1. 패키지명은 **`com.example.todoapp`** 이다. (`totoapp` 아님)
2. 데이터베이스명은 **`todolist_db`**, 테스트 DB는 **`todolist_db_test`** — 모두 소문자.
3. **엔티티를 컨트롤러에서 직접 반환하지 않는다.** 항상 DTO(Request/Response)로 변환한다.
4. **Todo 조회/수정/삭제 시 반드시 `user_id` 소유권을 검증한다.** 남의 리소스 접근은 404로 응답한다(존재 여부 노출 방지).
5. **삭제는 모두 Soft Delete**다. 물리 삭제(`DELETE FROM`) 금지. `deleted_at`을 채운다.
6. 예외 응답은 **`GlobalExceptionHandler` 한 곳**에서만 만든다. 컨트롤러에서 개별 try-catch로 응답을 조립하지 않는다.
7. 비밀번호는 **BCrypt**로만 저장한다. 평문/단방향 해시 직접 구현 금지.
8. Tiptap 본문 HTML은 **저장 전 서버에서 Jsoup으로 sanitize**하고, **렌더링 시 클라이언트에서 DOMPurify로 한 번 더** 거른다. 둘 중 하나만 하는 것은 금지.
9. **비밀/키를 코드에 하드코딩하지 않는다.** 모두 환경변수로 뺀다. `.env`, `application-local.yml`은 `.gitignore`에 넣는다.
10. **주석은 한글로 작성한다.** 단, 코드 식별자(클래스/변수/함수명)는 영문이다.
11. 임의로 라이브러리를 추가하지 않는다. 필요하면 먼저 이유와 함께 제안하고 승인을 받는다.

---

## 4. 코딩 컨벤션

### 공통
- 네이밍: DB는 `snake_case`, Java는 `camelCase`/`PascalCase`, TypeScript는 `camelCase`/`PascalCase`.
- API JSON 필드는 `camelCase`로 통일한다. (DB 컬럼 `created_at` → JSON `createdAt`)
- 날짜/시간은 서버·DB 모두 **UTC**로 저장하고, ISO-8601 문자열로 주고받는다. 표시 시점에만 로컬 타임존으로 변환한다.

### Java
- 계층: `controller` → `service` → `repository`. 컨트롤러가 리포지토리를 직접 호출하지 않는다.
- 패키지 구조는 **기능별(package-by-feature)**로 나눈다: `auth`, `user`, `todo`, `global`.
- DTO는 **record**로 만든다. 엔티티는 클래스.
- 엔티티에 `@Setter`를 열지 않는다. 상태 변경은 의미 있는 메서드로 표현한다 (`complete()`, `updateContent()`).
- 생성자 주입만 사용한다. 필드 주입(`@Autowired` 필드) 금지.
- `Optional`을 반환 타입으로만 쓰고, 필드나 파라미터로 쓰지 않는다.
- 조회 메서드에는 `@Transactional(readOnly = true)`를 붙인다.

### TypeScript
- `any` 금지. 불가피하면 `unknown` + 타입가드.
- `interface`는 객체 형태, `type`은 유니온/유틸리티에 사용한다.
- 서버 응답 타입은 `src/types/`에 한 번만 정의하고 재사용한다. 컴포넌트 안에서 인라인으로 다시 선언하지 않는다.
- 컴포넌트 파일명은 `PascalCase.tsx`, 그 외 유틸/훅은 `camelCase.ts`.
- 클라이언트 컴포넌트에만 `'use client'`를 붙인다. 기본은 서버 컴포넌트다.
- 데이터 패칭은 React Query 훅으로 통일한다. 컴포넌트 안에서 `fetch`를 직접 호출하지 않는다.

---

## 5. 인증 정책

### 토큰 (2종 구조)

| | Access Token | Refresh Token |
|---|---|---|
| 형식 | JWT | 불투명 랜덤 문자열 (JWT 아님) |
| 만료 | **30분** | **14일** |
| 저장 위치 | 프론트엔드 **localStorage** (키: `todo_access_token`) | **httpOnly + Secure + SameSite 쿠키** (`refresh_token`) |
| 전송 | `Authorization: Bearer <token>` 헤더 | 브라우저가 자동 전송 |
| 서버 저장 | 저장하지 않음 (stateless) | `refresh_tokens` 테이블에 **SHA-256 해시로** 저장 |

- **Refresh Token을 localStorage나 JS에서 접근 가능한 곳에 절대 두지 않는다.** 이것이 서버 로그아웃과 XSS 방어의 근거다.
- Refresh Token은 **회전(rotation)** 한다. 사용할 때마다 새로 발급하고 이전 것은 폐기한다.
- 이미 폐기된 Refresh Token이 다시 들어오면 **탈취로 간주해 해당 사용자의 모든 Refresh Token을 폐기**하고 재로그인시킨다.
- 프론트는 401(`TOKEN_EXPIRED`)을 받으면 자동으로 `/api/auth/refresh`를 1회 시도하고, 성공하면 원래 요청을 재시도한다. 실패하면 토큰을 지우고 `/login`으로 보낸다. **재시도 루프에 빠지지 않도록 refresh 요청 자체는 재시도 대상에서 제외한다.**
- 로그아웃은 서버 `POST /api/auth/logout`을 호출해 **Refresh Token을 DB에서 폐기하고 쿠키를 만료**시킨다. 클라이언트 토큰 삭제만으로 끝내지 않는다.
- 쿠키를 쓰므로 프론트의 모든 인증 요청은 `credentials: 'include'`, 백엔드 CORS는 `allowCredentials(true)` + **명시적 origin**(와일드카드 `*` 사용 불가)이다.
  - 로컬(`localhost:3000` ↔ `localhost:8080`)은 same-site이므로 `SameSite=Lax`로 동작한다.
  - 운영(Amplify 도메인 ↔ API 도메인)은 cross-site이므로 `SameSite=None; Secure`가 필요하다. 이 값은 프로파일별로 분리한다.

### 계정

- 회원가입 ID는 **이메일만** 허용, 비밀번호는 **6자 이상**.
- 소셜 로그인은 **Google OAuth2 (Spring Security OAuth2 Client, 백엔드 주도 리다이렉트)** 방식이다. NextAuth/Auth.js는 사용하지 않는다.
- 같은 이메일의 로컬 계정이 이미 있으면 구글 로그인을 **거부**한다(자동 연동하지 않음). `error=email_conflict`로 안내한다.

### 비밀번호 재설정

- 재설정 토큰은 랜덤 문자열이며, DB에는 **해시로만** 저장한다. 만료 **30분**, **1회용**.
- 재설정이 완료되면 해당 사용자의 **모든 Refresh Token을 폐기**한다.
- 존재하지 않는 이메일로 요청해도 **성공과 동일하게 응답**한다(계정 존재 여부 노출 방지).
- 메일 발송은 프로파일로 분리한다. 로컬은 재설정 링크를 **콘솔 로그로 출력**하고, 운영에서만 실제 SMTP를 사용한다. 로컬 개발을 위해 메일 서버를 띄우지 않는다.

---

## 6. 작업 방식

- **구현 전에 계획을 먼저 제시하고 승인을 받는다.** 승인 없이 대규모 파일 생성을 시작하지 않는다.
- 한 단계(Phase)가 끝나면 반드시 **완료 조건에 적힌 명령을 실제로 실행해서 통과하는지 확인**한 뒤 커밋한다.
- 커밋 메시지: `feat: ...`, `fix: ...`, `chore: ...`, `test: ...`, `docs: ...` (Conventional Commits, 본문은 한글 가능)
- 요구사항이 모호하면 **추측해서 만들지 말고 질문한다.**
- 기존 파일을 수정할 때는 관련 없는 코드를 함께 리팩터링하지 않는다.
- 라이브러리 API가 확실하지 않으면 **기억에 의존하지 말고 공식 문서를 확인한다.** 특히 Spring Boot 4 / Spring Security 7 / Next.js 16 / Tailwind CSS 4는 이전 메이저 버전과 문법이 다르므로, 학습된 지식으로 짐작하지 말 것.

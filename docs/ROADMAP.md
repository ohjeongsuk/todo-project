# ROADMAP — Todo List 프로젝트

> **버전** 1.8 · **최종 수정** 2026-08-28
> 이 문서는 "어떤 순서로 만드는가"를 정의하며, **완료 판정의 정본**이다.
> **한 번에 전체를 생성하지 않는다.** Phase 단위로 진행하고, 각 Phase의 DoD를 모두 만족한 뒤 다음으로 넘어간다.
> 기술 규칙은 `CLAUDE.md`, 기능 정의는 `PRD.md` 참조.

---

## 진행 현황

| Phase | 내용                       | 저장소   | 상태 |
| ----- | -------------------------- | -------- | ---- |
| 0     | 저장소 초기화              | 전체     | 🟡   |
| 1     | 백엔드 스캐폴딩            | backend  | ✅   |
| 2     | 도메인 & DB                | backend  | ✅   |
| 3     | 인증 (로컬) + 인증 테스트  | backend  | ✅   |
| 4     | Todo API + Todo 테스트     | backend  | ✅   |
| 5     | 구글 OAuth2 + OAuth 테스트 | backend  | ✅   |
| 6     | 프론트 스캐폴딩            | frontend | ✅   |
| 7     | 인증 화면                  | frontend | ⬜   |
| 8     | Todo 화면                  | frontend | ⬜   |
| 9     | 인터랙션 다듬기            | frontend | ⬜   |
| 10    | 전체 검증                  | 전체     | ⬜   |
| 11    | AWS 배포                   | 전체     | ⬜   |

⬜ 대기 · 🟡 진행중 · ✅ 완료

> **Phase 0이 🟡인 이유 (2026-09-01):** DoD 9개 중 8개가 통과했고 **원격 푸시 하나만 남았다.** 세 저장소 모두 GitHub 원격이 연결돼 있으나 `main`·`develop`이 push되지 않았다. 푸시하면 Phase 0은 ✅가 된다.

> ### 스캐폴딩 정합성 점검 (2026-08-28)
>
> 스캐폴딩이 스펙과 어긋난 채 진행되어 아래를 바로잡았다. **해결됨** 항목은 재발 방지 DoD가 각 Phase에 들어가 있다.
>
> | 항목                                   | 발견 당시                                                                       | 결과                                                                                                                                                                                         |
> | -------------------------------------- | ------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
> | `todo-frontend`가 Next.js 16.3.3       | Amplify SSR 지원 범위(12~15) 밖이라 Phase 11 배포 불가 상태였음                 | ❌ **기록 오류 — 마이그레이션되지 않았다.** 실제 `package.json`은 `next` 16.3.3 / `eslint-config-next` 16.3.3이다. 2026-09-01 **16.3.3 유지로 확정**(`CLAUDE.md` 3장 스택 표·`docs/guides` 5개 문서가 모두 16 기준). 배포 리스크는 Phase 11 항목으로 이관                                                                                            |
> | 프론트에 `src/` 없음                   | `app/`·`components/`·`lib/`가 루트 직하                                         | ✅ **`src/` 아래로 이동 (2026-09-01 Phase 6에서 실제 수행).** 2026-08-28 시점의 완료 기록은 오류였다 — `git log` 커밋 3건 어디에도 없었다. `tsconfig` paths(`@/*` → `./src/*`), `components.json` css 경로 동시 수정                                                                                       |
> | `layout.tsx`가 `LayoutProps<"/">` 사용 | Next 16 전역 타입이라 15에서 컴파일 실패                                        | ➖ **불필요해짐.** Next 16 유지 결정으로 `LayoutProps<"/">`가 정상 타입이 됐다. 실제로 교체된 적도 없었다(기록 오류)                                                                                                                                             |
> | `eslint.config.mjs`가 16 방식          | 15의 `eslint-config-next`는 flat config를 직접 내보내지 않음                    | ➖ **불필요해짐.** Next 16 유지이므로 `eslint-config-next` 16의 flat config를 그대로 쓴다                                                                                                                                                                |
> | `AGENTS.md`                            | Next 16의 `next dev`가 자동 생성한 파일. 15에서는 재생성되지 않고 내용도 부정확 | ⚠️ **삭제로는 해결되지 않는다.** 파일 본문이 밝히듯 `next dev`가 재생성한다(`node_modules/next/dist/server/lib/generate-agent-files.js`). Next 16 유지이므로 생성물로 인정해 커밋하고, 저장소용 `CLAUDE.md`를 `@AGENTS.md` → `@../CLAUDE.md` 임포트로 교체했다                                                                                                                         |
> | 워크스페이스 루트 오인                 | 부모 `todo-project`에도 `package-lock.json`이 있어 Next가 루트를 잘못 추론      | ✅ **2026-09-01 Phase 6에서 실제 지정.** 2026-08-28 완료 기록은 오류였다 — `next.config.ts`는 빈 객체였다. **원인이던 루트 npm 파일도 삭제**됐으나, 재발 시 조용히 어긋나므로 설정은 유지한다                                                  |
> | `pom.xml`에 `springdoc` 없음           | Phase 1 DoD "Swagger UI 접속" 불가                                              | ✅ **3.1.0 핀.** 이 버전의 부모가 `spring-boot-starter-parent` 4.1.0이라 Boot 4.1.x에 대응                                                                                                   |
> | `pom.xml`에 `jsoup` 없음               | Phase 4 XSS 정화 불가                                                           | ✅ **1.23.2 핀**                                                                                                                                                                             |
> | JWT 라이브러리 미선정                  | `CLAUDE.md` 3장 표에 JWT 라이브러리가 명시되어 있지 않음                        | ✅ **jjwt 0.12.6**(`jjwt-api`/`jjwt-impl`/`jjwt-jackson`)이 `pom.xml`에 핀되어 있고, **`CLAUDE.md` 3장 스택 표와 「버전 관련 확정 사항」에 등재됐다**(v1.8). Phase 3은 이 버전을 그대로 쓴다 |
> | `todo-backend/.gitattributes`          | Git Bash에서 `./mvnw` `bad interpreter` 위험                                    | ✅ 존재함 (`/mvnw text eol=lf`, `*.cmd text eol=crlf`)                                                                                                                                       |
> | 백엔드 버전                            | Spring Boot 4.1.1 + `java.version` 21                                           | ✅ 스펙 일치. 변경 없음                                                                                                                                                                      |
>
> #### 아직 남은 것 (2026-08-28 재확인)
>
> | 항목                     | 현재 상태                                                                                                                                                                                                                                                                                                                         | 처리 Phase |
> | ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- |
> | 커밋·원격                | ✅ **부분 해소 (2026-09-01 확인).** 세 저장소 모두 커밋이 쌓였고 GitHub 원격(`ohjeongsuk/todo-{project,backend,frontend}`)이 연결됐다. **다만 `main`·`develop` 어느 것도 아직 푸시되지 않았다** — 원격에 브랜치가 없다(`todo-frontend`에만 낡은 `origin/master` 참조가 남아 있다)                                            | Phase 0    |
> | ~~브랜치명~~             | ✅ **해소.** 세 저장소 모두 현재 브랜치가 `main`이고 `develop`이 존재하며 로컬 `master`는 없다                                                                                                                                                                                                                                    | —          |
> | ~~루트 `.gitignore`~~    | ✅ **해소.** `node_modules/` 재발 방지 규칙을 추가했다                                                                                                                                                                                                                                                                            | —          |
> | ~~백엔드 `.gitignore`~~  | ✅ **해소.** `.env` / `.env.*` / `!.env.example` 3줄이 있고 `git check-ignore` 종료코드로 확인했다(`.env`=0, `.env.example`=1)                                                                                                                                                                                                     | —          |
> | ~~프론트 `.gitignore`~~  | ✅ **해소.** `!.env.example` 예외를 추가했고 종료코드로 확인했다                                                                                                                                                                                                                                                                  | —          |
> | ~~문서 경로~~            | ✅ **해소.** `docs/`를 정본으로 확정하고 `CLAUDE.md` 2장 구조도와 상단 참조 경로를 정정했다 (v1.8)                                                                                                                                                                                                                                | —          |
> | ~~루트 npm 파일~~        | ✅ **해소.** 루트 `package.json`·`package-lock.json`·`node_modules/`를 삭제했다. `shadcn`은 `todo-frontend/package.json`에 이미 있어 기능 손실이 없고, `.mcp.json`의 shadcn 서버는 `npx shadcn@latest mcp`라 루트 설치에 의존하지 않는다                                                                                          | —          |
> | ~~`docs/guides/`~~       | ✅ **해소.** 5개 전부 다른 프로젝트에서 넘어온 문서였고(존재하지 않는 "PRD 1.3 기술 스택" 참조, 모노레포·Next 16·없는 npm 스크립트 전제), **이 프로젝트 기준으로 전부 재작성**했다. `nextjs-16.md`→`nextjs-app-router.md`, `forms-react-hook-form.md`→`forms.md`로 개명. `README.md`를 추가해 "참고 자료이며 `CLAUDE.md`가 우선"임을 명시 | —          |
> | ~~폼 라이브러리 미결정~~ | ✅ **해소.** `CLAUDE.md` 3장에 **"라이브러리를 쓰지 않는다 — `useState` + 수동 검증"**으로 확정. `npx shadcn add form` 금지(=`react-hook-form` 유입 경로)와 Tiptap dirty 판정 주의를 함께 명시                                                                                                                                    | —          |
> | 백엔드 설정 파일         | `src/main/resources/application.properties` 하나뿐. `application.yml` + `-local` + `-prod` 분리 미완                                                                                                                                                                                                                              | Phase 1    |
> | 백엔드 문서·예시         | `.env.example`, 저장소용 `CLAUDE.md` 없음                                                                                                                                                                                                                                                                                         | Phase 1    |
> | ~~프론트 마감 작업~~     | ✅ **해소 (2026-09-01 Phase 6).** 디자인 토큰·다크 전략·`new-york` 스타일·`.env.example`·패키지 7종 전부 처리하고 실측 검증했다                                                                                                                                                                                              | —          |

> **테스트는 마지막에 몰아 쓰지 않는다.** 기능을 만든 Phase에서 함께 작성해 그 Phase의 DoD로 삼는다. Phase 10은 새 테스트를 쓰는 단계가 아니라 전체를 확인하는 단계다.

---

## 요구사항 ↔ Phase 추적표

> `PRD.md` 3장의 P0 요구사항이 **어느 Phase에서 구현되고 어느 Phase에서 검증되는지**의 정본이다.
> **여기에 행이 없는 P0는 구현되지 않는다.** `PRD.md` 3장에 요구사항을 추가하면 이 표에 먼저 행을 넣고, 해당 Phase의 작업·DoD에 실제로 기술한다.

| ID                                       | 구현 Phase                                                    | 검증 Phase (DoD)                                 |
| ---------------------------------------- | ------------------------------------------------------------- | ------------------------------------------------ |
| AUTH-01 회원가입                         | 3(API) · 7(화면)                                              | 3 · 7 · 10                                       |
| AUTH-02 비밀번호 6자 이상 + 72바이트     | 3(바이트 validator) · 7(실시간 검증)                          | 3(한글 25자 → 400) · 7(제출 전 인라인 안내) · 10 |
| AUTH-03 이메일 중복                      | 3(409) · 7(인라인 문구)                                       | 3 · 7                                            |
| AUTH-04 로그인·JWT 24h                   | 3 · 7                                                         | 3 · 7 · 10                                       |
| AUTH-05 구글 로그인                      | 5 · 7(`/oauth/callback`)                                      | 5 · 7 · 10                                       |
| AUTH-06 로그아웃                         | 7 (프론트 전용, 서버 API 없음)                                | 7 · 10                                           |
| AUTH-07 라우트 보호                      | 7 (`(main)` 클라이언트 레이아웃)                              | 7(만료 토큰 케이스) · 10                         |
| AUTH-08 헤더 닉네임 / 이메일 미표시      | 3(`/auth/me` 응답) · 7(헤더)                                  | 3(응답 필드) · 7(DOM에 이메일 없음) · 10         |
| AUTH-09 계정 충돌 거부                   | 5 · 7(안내 문구)                                              | 5 · 7 · 10                                       |
| TODO-01 제목 필수·200자                  | 4 · 8                                                         | 4 · 10                                           |
| TODO-02 Tiptap 본문                      | 4(정화·50,000자) · 8(에디터)                                  | 4 · 8 · 10                                       |
| TODO-03 우선순위                         | 4 · 8                                                         | 4 · 8 · 10                                       |
| TODO-04 마감일                           | 4 · 8                                                         | 4 · 8 · 10                                       |
| TODO-05 생성일 내림차순                  | 4 · 8                                                         | 4 · 8 · 10                                       |
| TODO-06 10건 페이지네이션                | 4 · 8                                                         | 4 · 8 · 10                                       |
| TODO-07 완료 필터                        | 4 · 8                                                         | 4 · 8 · 10                                       |
| TODO-08 제목 검색                        | 4 · 8                                                         | 4 · 8 · 10                                       |
| TODO-09 상세(항상 편집)                  | 4 · 8                                                         | 8 · 10                                           |
| TODO-10 저장(완료 상태 불변)             | 4(PUT에 `completed` 없음) · 8                                 | 4 · 8 · 10                                       |
| TODO-11 완료 토글 즉시 반영              | 4(멱등 toggle) · 9(낙관적 업데이트)                           | 4 · 9(연타 수렴값) · 10                          |
| TODO-12 삭제 즉시 반영 / Soft Delete     | 4 · 8(상세→목록 이동) · 9(낙관적 제거)                        | 4 · 8 · 9 · 10                                   |
| TODO-13 실패 시 롤백·알림                | **8(저장 실패: 폼 유지 + 에러)** · 9(토글·삭제 롤백 + 토스트) | 8 · 9 · 10                                       |
| TODO-14 타인 리소스 차단                 | 4(404) · 8(전용 화면)                                         | 4 · 8 · 10                                       |
| TODO-15 XSS 이중 방어                    | 4(Jsoup) · 6(`sanitize.ts`) · 8(`setContent` 직전)            | 4 · 8 · 10                                       |
| TODO-16 이탈 확인                        | 8 (3계층)                                                     | 8                                                |
| UX-01 스켈레톤                           | 6(컴포넌트) · 7(인증 판정·콜백) · 8(목록·상세)                | 7 · 8 · 10                                       |
| UX-02 빈 상태                            | 6 · 8                                                         | 8 · 10                                           |
| UX-03 검색 결과 없음                     | 8                                                             | 8 · 10                                           |
| UX-04 에러 + 재시도 버튼                 | 6(`ErrorState.onRetry` 필수) · 7 · 8                          | 6 · 7 · 8(재요청 확인) · 10                      |
| UX-05 반응형 360~1920                    | 6(토큰·컨테이너) · 8                                          | 8(360px) · 10(1920px)                            |
| UX-06 label·키보드                       | 7 · 8                                                         | 7 · 8 · 10                                       |
| UX-07 다크 토큰 (`prefers-color-scheme`) | 6                                                             | 6 · 10                                           |

> `PRD.md` 5.1의 **에러 문구 매핑 표**는 Phase 6에서 `lib/errorMessages.ts`로 단일화하고, Phase 7·8에서 화면별로 적용, Phase 10에서 6종 전부를 대조한다.
> `PRD.md` 7장 비기능 요구사항의 검증 위치(응답 속도 4 · 체감 반응 9 · 브라우저 10 · 반응형 8·10 · 인증 보안 3·4 · 시크릿 관리 10·11 · XSS 4·8 · 문서화 4 · 접근성 7·8)는 위 표 및 각 Phase DoD와 일치한다.

---

## Phase 0 — 저장소 초기화

**목표**: 폴리레포 3개 저장소를 만들고 문서를 자리잡게 한다.

> ⚠️ **폴리레포다.** `todo-project`(문서)·`todo-backend`·`todo-frontend`가 각각 독립 git 저장소이며 하나의 커밋으로 묶이지 않는다. 2026-09-01에 `CLAUDE.md` 1장이 "저장소는 하나(모노레포)"라고 적고 있던 것을 실제 구조에 맞게 정정했고, `PRD.md` 8장·`CHECKLIST.md` 1.6·`shrimp-rules.md`·에이전트 정의 2건도 함께 고쳤다. 특히 `CHECKLIST.md` 1.6은 **`todo-frontend/.git`이 없어야 한다**는 반대 기준이라 Phase 10에서 무조건 실패했을 항목이다.

**작업**

- `todo-project/` 생성 후 `git init`
- `CLAUDE.md`, `PRD.md`, `ROADMAP.md` 배치
- `.gitignore`에 `todo-backend/`, `todo-frontend/` 추가 (**필수**)
- **루트 `.gitignore`에 `node_modules/` 추가** — 재발 방지용이다
  > 루트의 `package.json`·`package-lock.json`·`node_modules/`는 **이미 삭제했다.** `shadcn`은 `todo-frontend`에 있고 `.mcp.json`은 `npx`로 받아 쓰므로 문서 저장소에 npm 파일을 두지 않는다. 다만 루트에서 `npx shadcn`을 실수로 실행하면 `node_modules/`가 다시 생기므로 무시 규칙은 남긴다.
- `todo-backend/`, `todo-frontend/` 폴더 생성 후 각각 `git init`
- 각 저장소 `.gitignore`에 `.env*` + **`!.env.example`** (예외 줄이 없으면 예시 파일까지 무시됨)
  > 현재 백엔드는 `.env*` 규칙 자체가 없고, 프론트는 `.env*`만 있고 `!.env.example` 예외가 없다. **양쪽 모두 손봐야 한다.**
- **브랜치 정리** — 세 저장소 모두 `master`다. 첫 커밋 시 `git branch -m main`(또는 `git init -b main`)으로 `main`을 만들고, `main`에서 `develop`을 분기한다. 이후 작업은 `feature/{작업명}` → `develop` → `main` (`CLAUDE.md` 2장)
- ~~문서 경로 확정~~ — ✅ **완료(v1.8).** `docs/`를 정본으로 확정했다. `CLAUDE.md`만 루트에 두는데, Claude Code가 상위 디렉토리를 거슬러 올라가며 자동 로드하는 대상이 `CLAUDE.md`이기 때문이다. 2장 구조도와 상단 참조 경로를 정정했다
- ~~`nextjs-16.md` 처리~~ — ✅ **완료.** 삭제하고 **`docs/guides/nextjs-app-router.md`로 재작성**했다(ROADMAP이 `nextjs-15.md`라 적고 있던 것을 실제 파일명으로 정정. `.claude/agents/dev/nextjs-app-developer.md`가 없는 `nextjs-15.md`를 임포트하고 있던 것도 함께 고쳤다). 단순 버전 치환이 아니라 이 프로젝트 기준으로 다시 썼다: "Server Components 우선"을 **"클라이언트 컴포넌트 우선"**으로 뒤집고(토큰이 localStorage에 있어 서버 페칭이 불가능), Next 16 전용 내용(`proxy.ts`, `cacheComponents`, 최상위 `typedRoutes`)을 걷어내고, 쓰지 않는 기능(Server Actions·Route Handlers·Parallel/Intercepting Routes·ISR·`notFound()`)을 **금지 표로 명시**했다
- ~~나머지 가이드 4개 처리~~ — ✅ **완료.** 넷 다 이 프로젝트 기준으로 재작성했다. `component-patterns`(서버 우선 → **클라이언트 우선**), `styling-guide`(`next-themes` 토글 → **미디어쿼리 다크 모드**), `project-structure`(모노레포 → **폴리레포**), `forms-react-hook-form.md` → **`forms.md`**(라이브러리 없는 폼). `guides/README.md`를 추가해 지위(참고 자료, `CLAUDE.md` 우선)와 교체 이력을 남겼다
  > ⚠️ 옛 `forms-react-hook-form.md`는 `react-hook-form`·`zod`가 **"이미 설치되어 있다"고 서술**했고, 비밀번호 규칙을 **"6자 이상이 전부"**로 적어 UTF-8 72바이트 한계를 누락하고 있었다. 그대로 뒀다면 Phase 7에서 `AUTH-02`가 조용히 깨졌을 문서다.
- GitHub에 원격 저장소 3개 생성 및 연결

**DoD**

- [x] 부모 저장소에서 `git status` 시 하위 폴더가 나타나지 않음 — `git status --porcelain -uall`이 빈 출력
- [x] **부모 저장소에서 `git status` 시 `node_modules/`가 나타나지 않음** — 루트 `.gitignore`에 규칙 존재
- [x] **세 저장소 각각에서 `.env`는 무시되고 `.env.example`은 무시되지 않음** — 세 저장소 모두 `git check-ignore -q .env`가 종료코드 `0`, `.env.example`이 `1`
  > ⚠️ **`-v` 출력만 보고 판정하지 않는다.** 부정 규칙(`!.env.example`)에 매칭되면 `-v`는 그 규칙을 출력하지만 종료코드는 `1`(무시되지 않음)이다. 출력 유무가 아니라 **종료코드**로 판정한다.
- [x] 세 저장소 모두 현재 브랜치가 `main`이고 `develop`이 존재함 (로컬 `master` 없음)
- [ ] 세 저장소 모두 첫 커밋 및 원격 푸시 완료 — **커밋은 완료, 푸시는 미완료.** GitHub 원격은 세 저장소 모두 연결됐으나 `main`·`develop` 어느 것도 push되지 않아 원격에 브랜치가 없다. `todo-frontend`에만 낡은 `origin/master` 참조가 남아 있어 정리가 필요하다

  > **푸시 절차 (2026-09-01 기준, 아직 실행하지 않음).** 로컬에 쌓인 커밋은 `todo-project` main 14 / develop 4, `todo-backend` main 10 / develop 2, `todo-frontend` main 8 / develop 3건이다.
  >
  > ```bash
  > # 세 저장소 각각에서
  > git push -u origin main
  > git push -u origin develop
  > ```
  >
  > ⚠️ **`todo-frontend`는 순서가 중요하다.** 원격에 낡은 `master`(`f1a0384`, 현재 `main`의 조상)가 있고 GitHub 기본 브랜치가 거기에 잡혀 있을 가능성이 크다. **기본 브랜치인 채로는 삭제가 거부되므로** 다음 순서를 지킨다:
  >
  > 1. `git push -u origin main`
  > 2. GitHub → Settings → General → Default branch를 **`main`으로 변경**
  > 3. 그 다음에야 `git push origin --delete master`
  >
  > 순서를 거꾸로 하면 3에서 막힌다. 2는 웹 UI 설정 변경이라 사용자가 직접 한다.
  >
  > 푸시 전 확인: `gh` CLI가 설치돼 있지 않아 세 저장소가 public인지 private인지 확인하지 못했다. **public이면 푸시 즉시 코드가 공개된다.** `.env`·`application-local.yml`은 무시 규칙에 걸려 있음을 종료코드로 확인했으나, 이미 커밋된 파일에 비밀값이 없는지는 푸시 직전에 한 번 더 본다.
- [x] 첫 커밋의 파일 수가 예상 범위 안임 — `todo-project` 10개 / `todo-backend` 11개 / `todo-frontend` 19개
- [x] `CLAUDE.md` **1장** 구조도의 문서 경로가 실제 파일 위치와 일치함 — 2026-09-01 정정. `PRD.md`·`ROADMAP.md`·`DEV_TOOLS.md`·`guides/`가 빠져 있었고, 없는 `API.md`·`DESIGN.md`가 있는 것처럼 적혀 있었다(둘은 미작성 표시로 남겼다). 구조도가 2장이 아니라 1장에 있다는 점도 함께 정정했다
- [x] `docs/guides/`가 `README.md` + 재작성된 5개 문서로만 구성됨 — `component-patterns.md`, `forms.md`, `nextjs-app-router.md`, `project-structure.md`, `styling-guide.md`. 금지 파일 2종 없음
- [x] **`todo-frontend/package.json`에 `react-hook-form`·`zod`·`@hookform/resolvers`·`next-themes`가 없음** — 4종 전부 부재 확인

---

## Phase 1 — 백엔드 스캐폴딩

**저장소**: `todo-backend`

**작업**

- Spring Initializr 기준 프로젝트 생성 (Spring Boot 4.x, JDK 21, Maven, `com.example`)
- 의존성: Web, Data JPA, Security, Validation, PostgreSQL Driver, OAuth2 Client, Lombok, **Jsoup**(HTML 정화), **jjwt**(JWT 생성·검증)
- **SpringDoc OpenAPI는 `springdoc-openapi-starter-webmvc-ui` 3.x를 명시한다.** 2.8.x는 Spring Boot 3.x 전용이라 기동에 실패한다
- 패키지 골격 생성: `domain / service / controller / dto / config / exception`
- **`application.properties`를 삭제하고 `application.yml`로 교체한다** (현재 저장소에는 `.properties`만 있다). 이어 `application-local.yml` + **`application-prod.yml`** 분리, 환경변수 바인딩 (프로파일별 `ddl-auto`는 `CLAUDE.md` 12장 표 참조)
  > ⚠️ 둘을 동시에 두면 `.properties`가 `.yml`보다 우선 적용되어, `.yml`에 적은 설정이 조용히 무시된다.
- **`application.yml`에 `spring.profiles.active: local` 기본값 지정** (없으면 DB 정보 없이 기동을 시도해 실패)
- `.gitignore`(**`.env*` + `!.env.example` 추가** — 현재 Initializr 기본값이라 `.env` 규칙이 없다), `.env.example`, 저장소용 `CLAUDE.md` 작성 (상단에 `@../CLAUDE.md` 임포트)
- `.gitattributes`는 이미 있다(`/mvnw text eol=lf`). **삭제하거나 덮어쓰지 않는다** (`CLAUDE.md` 13장)

**DoD**

- [x] `pom.xml`의 SpringDoc이 **현재 Boot 버전에 대응하는 정확한 3.x 버전으로 핀**되어 있음 (범위 지정 금지 — `CLAUDE.md` 3장) — `springdoc-openapi-starter-webmvc-ui:3.1.0`
- [x] `pom.xml`에 **`jsoup`이 포함**되어 있음 (Phase 4의 XSS 정화 전제) — `jsoup:1.23.2`
- [x] `pom.xml`에 **jjwt 3종(`jjwt-api`/`jjwt-impl`/`jjwt-jackson`)이 동일 버전으로 핀**되어 있음 (Phase 3의 `JwtTokenProvider` 전제) — 모두 `0.12.6`
- [x] `./mvnw dependency:tree` 오류 없음 (2026-08-31 재확인: `BUILD SUCCESS`)
- [x] **`src/main/resources`에 `application.properties`가 없고 `application.yml`·`application-local.yml`·`application-prod.yml`이 있음**
- [x] `./mvnw spring-boot:run`이 옵션 없이 local 프로파일로 기동 성공 (2026-08-31 재확인)
- [x] **기동 로그에 `The following 1 profile is active: "local"`이 찍힘**
- [x] `http://localhost:8080/swagger-ui/index.html`이 200으로 열리고 API 목록 화면이 렌더됨
- [x] `git check-ignore -v .env`가 규칙에 매칭되고 `.env.example`은 매칭되지 않음

> ### Phase 1 재검증 기록 (2026-08-31)
>
> DoD를 실제로 재실행해 검증하는 과정에서 **애플리케이션 기동 자체가 실패하는 회귀**를 발견하고 해결했다.
>
> | 항목 | 발견 당시 | 결과 |
> | --- | --- | --- |
> | Jackson 2→3 전환 미반영 | Spring Boot 4.1의 Jackson 자동설정 기본값이 `com.fasterxml.jackson.*`(2.x)에서 `tools.jackson.*`(3.x, `JsonMapper`)로 바뀌었는데, `JwtAuthenticationEntryPoint`·`JwtAccessDeniedHandler`가 여전히 `com.fasterxml.jackson.databind.ObjectMapper`(Jackson 2.x, `jjwt-jackson`의 전이 의존성으로 클래스패스엔 있으나 Spring이 빈으로 등록하지 않음)를 생성자로 요구해 `spring-boot:run` 기동이 `UnsatisfiedDependencyException`으로 전면 실패했다(컴파일은 항상 성공했으므로 발견이 늦어짐). | ✅ 두 클래스의 import를 `tools.jackson.databind.ObjectMapper`로 교체(다형성으로 `JsonMapper` 빈이 정상 주입됨). 재기동 후 Swagger UI·`/v3/api-docs` 200 확인 |
>
> ⚠️ Spring Boot 4.1 이후 Jackson 관련 코드를 새로 작성할 때는 `com.fasterxml.jackson.*`이 아니라 **`tools.jackson.*`**를 사용한다. `com.fasterxml.jackson`은 `spring-boot-jackson2`(deprecated) 모듈을 별도로 추가하지 않는 한 Spring이 자동으로 빈을 등록하지 않는다.

> SpringDoc 버전은 `CLAUDE.md` 3장에서 3.x로 확정됐다. 다시 조사하거나 2.x로 되돌리지 않는다.

---

## Phase 2 — 도메인 & DB

**저장소**: `todo-backend`

**작업**

- `createdb todolist_db` · `createdb todolist_db_test`
- `BaseEntity` (`@MappedSuperclass`, JPA Auditing)
- `User`, `Todo` 엔티티 + `Priority` enum + `AuthProvider` enum
- **`Todo.user`는 `@ManyToOne(fetch = FetchType.LAZY)`** (기본값 EAGER 금지)
- **`hibernate.jdbc.time_zone: UTC` 설정** — 로컬 KST와 RDS UTC의 9시간 어긋남 방지
- 입력값 제약을 스키마와 일치시킨다 (`CLAUDE.md` 4장 제약 표)
- `UserRepository`, `TodoRepository` (`deleted_at IS NULL` 조건 포함 쿼리)
- 인덱스 `idx_todos_user_deleted`
- `src/test/resources/application-test.yml` (todolist_db_test, `ddl-auto: create-drop`)
- **`@EnableJpaAuditing`은 메인 애플리케이션 클래스에 붙인다** (`@Configuration`에 두면 `@DataJpaTest`가 로드하지 않아 `created_at`이 null이 됨)
- Repository 테스트에 **`@AutoConfigureTestDatabase(replace = NONE)` + `@ActiveProfiles("test")`** 필수 (없으면 임베디드 DB로 교체 시도)

**DoD**

- [x] 애플리케이션 기동 시 `users`, `todos` 테이블 자동 생성 — 실측: `POST /api/auth/signup` → `POST /api/todos` 호출이 각각 200으로 성공(신규 Todo id 10103), `todolist_db_test`는 `ddl-auto: create-drop`이라 Repository 테스트마다 빈 스키마에서 테이블을 새로 만들고 버리는 것으로 매번 검증됨
- [x] `created_at`, `updated_at` 자동 기록 확인 — 실측: 방금 생성한 Todo 응답의 `createdAt`/`updatedAt`이 둘 다 null이 아니고 동일 시각(신규 생성이므로)으로 채워짐
- [x] Repository 단위 테스트(`@DataJpaTest`) 통과 — **통합 테스트 번호 체계(1~8)와 별개** — `UserRepositoryTest` 4건 + `TodoRepositoryTest` 4건, 총 8건 통과
- [x] 테스트가 `todolist_db_test`를 바라보고 실행됨 (H2 사용하지 않음) — 실측: 테스트 로그의 `Database JDBC URL`이 `jdbc:postgresql://localhost:5432/todolist_db_test`, H2 언급 없음
- [x] `@DataJpaTest`에서 `created_at`이 null이 아님 (Auditing 정상 동작) — `UserRepositoryTest.createdAt이_자동으로_기록된다`/`TodoRepositoryTest` 동일 테스트로 확인
- [x] 저장된 `created_at`이 UTC 기준임 (KST로 9시간 밀리지 않음) — `UserRepositoryTest.createdAt이_UTC_기준으로_저장된다` 통과. API 실측으로도 재확인: Todo 생성 응답 `createdAt`(`09:02:54`)이 `date -u`로 잰 실제 UTC 시각(`09:02:59`)과 5초 이내 일치, KST(9시간 차이)로 밀리지 않음

> ⚠️ Phase 2와 무관하게 발견: 한글이 포함된 JSON 본문을 curl(Git Bash, 로컬 콘솔 인코딩)로 보내면 서버가 `HttpMessageNotReadableException`(잘못된 UTF-8)을 `GlobalExceptionHandler`에서 400이 아닌 500 `INTERNAL_ERROR`로 응답한다. 클라이언트 요청 자체가 깨진 경우이므로 서버 책임은 아니지만, 400으로 내려주는 편이 API 계약상 더 정확하다 — Phase 3~4 범위 밖이라 별도 이슈로만 기록해 둔다.

전체 테스트 스위트(19건) 회귀 없음. 검증 중 로컬 DB에 남은 흔적: `phase2-check@example.com` 계정(로컬 전용), Todo id 10103 — 실제 서비스 데이터가 아니므로 정리하지 않음.

---

## Phase 3 — 인증 (로컬) + 인증 테스트

**저장소**: `todo-backend` · **관련 요구사항**: AUTH-01~04, 07~08 (AUTH-06 로그아웃은 프론트 전용이라 Phase 7)

**작업**

- `SecurityConfig` (SecurityFilterChain, CORS, 경로별 인가)
  - **`permitAll` 경로에 Swagger(`/swagger-ui/**`, `/v3/api-docs/**`)를 반드시 포함** (`CLAUDE.md` 6장 목록 그대로)
  - CORS 허용 헤더에 `Authorization`, `Content-Type` 명시
- `JwtTokenProvider` (생성/검증, 24시간 만료, **`sub = user.id`**)
- `JwtAuthenticationFilter` (`sub` → id 조회, `deleted_at IS NULL` 확인)
- `BCryptPasswordEncoder` 빈
- `AuthController`: signup / login / me (**로그아웃 API는 만들지 않는다**)
- DTO: `SignupRequest`, `LoginRequest`, `TokenResponse`, `UserResponse`
- `GlobalExceptionHandler` + `ErrorCode` + **공통 응답 `ApiResponse<T>`**
- **Swagger `@SecurityScheme`(bearerAuth) 설정** — Authorize 버튼으로 보호된 API 호출 가능하게
- **`AuthenticationEntryPoint` + `AccessDeniedHandler` 구현** — Security 필터 단계의 401/403도 `ApiResponse` 포맷으로 응답 (`GlobalExceptionHandler`는 필터 예외를 잡지 못함)
- **통합 테스트 1~3번 작성**

**DoD**

- [x] **`POST /api/v1/auth/signup`이 403이 아니라 200/409로 응답** (CSRF 비활성화 확인 — 안 하면 여기서 전부 막힌다)
- [x] **응답에 `Set-Cookie: JSESSIONID`가 없음** (`STATELESS` 세션 정책 확인)
- [x] 회원가입 → 사용자 생성 + JWT 반환
- [x] **한글 25자 비밀번호로 가입 시 500이 아니라 400 `INVALID_INPUT`** (BCrypt 72바이트 한계 — `CLAUDE.md` 4장)
- [x] 중복 이메일 시 409 `EMAIL_DUPLICATED`
- [x] 로그인 성공 시 유효 JWT, 실패 시 401
- [x] **미가입 이메일로 로그인한 401과 비밀번호가 틀린 401의 `code`·`message`가 완전히 동일함** (계정 존재 여부를 구분 노출하지 않는다 — `PRD.md` 5.1)
- [x] 토큰 없이 `/api/v1/auth/me` 호출 시 401
- [x] `/api/v1/auth/me` 응답에 `nickname`과 `email`이 모두 포함됨 (AUTH-08 — 화면 표시 여부는 Phase 7에서 통제)
- [x] 비밀번호가 DB에 해시로 저장됨
- [x] **Security 도입 후에도 Swagger UI 접속 가능** (Phase 1 DoD 회귀 방지)
- [x] Swagger Authorize에 토큰을 넣고 `/auth/me` 호출이 200으로 성공
- [x] 모든 응답이 `{success, data, error}` 포맷을 따름
- [x] **토큰 없이 호출한 401 응답도** `{success:false, error:{code:"UNAUTHORIZED"}}` 포맷임
- [x] **인증 통합 테스트 3건 통과**

> ### Phase 3 구현 기록 (2026-08-31)
>
> 착수 시점에 `AuthService`·`SecurityConfig`·`JwtTokenProvider`·`RefreshTokenCookieFactory`·`GlobalExceptionHandler` 등 서비스·보안 레이어는 이미 완성되어 있었으나(CLAUDE.md 5장 refresh token 회전 정책까지 반영), 이를 호출할 `AuthController`가 없어 인증 API를 전혀 쓸 수 없는 상태였다. 이번 Phase에서 `AuthController`(signup/login/refresh/me)와 `OpenApiConfig`(bearerAuth)를 신규 작성해 기존 자산을 그대로 배선했다.
>
> - `POST /api/auth/refresh`는 ROADMAP 작업 목록에 명시되어 있지 않았지만, `SecurityConfig`에 이미 permitAll 경로로 등록되어 있고 `AuthService.refresh()`·`RefreshTokenService.rotate()`도 이미 구현되어 있어 함께 작성했다(사용자 승인). curl로 토큰 회전과, 이미 폐기된 토큰 재사용 시 401로 거부되는 탈취 감지 로직까지 확인했다.
> - `AuthControllerTest`는 `@SpringBootTest` + `@AutoConfigureMockMvc` + `@ActiveProfiles("test")` + `@Transactional`로 작성해 `todolist_db_test`를 사용했다(H2 미사용). Boot 4.1에서 `@AutoConfigureMockMvc`의 패키지가 `org.springframework.boot.webmvc.test.autoconfigure`로 이동한 것을 로컬 jar 직접 검색으로 확인했다.

---

## Phase 4 — Todo API + Todo 테스트

**저장소**: `todo-backend` · **관련 요구사항**: TODO-01~15

**작업**

- `TodoController` 6개 엔드포인트 (목록/생성/단건/수정/토글/삭제)
- **`PUT`은 전체 교체이며 `TodoUpdateRequest`에 `completed`를 넣지 않는다**
- **`PATCH /toggle`은 바디로 `{"completed": true}`를 받는다** (서버가 뒤집지 않음)
- `TodoService`: 소유권 검증, Soft Delete, **HTML 정화**
- **HtmlSanitizer**: Jsoup Safelist, 허용 태그·`rel` 주입·스킴 제한 (`CLAUDE.md` 6장)
- 페이지네이션 + `completed`(미지정 시 전체) + `keyword`(대소문자 무시) 필터
- 정렬은 `createdAt,desc` 고정. **허용 필드 화이트리스트(`createdAt`, `dueDate`) 밖의 값은 기본값으로 대체** (없는 프로퍼티로 500 방지)
- `PageResponse<T>` DTO → **`ApiResponse.data` 안에 담아 반환**
- **`TodoResponse`에 사용자 정보를 넣지 않는다** (본인 데이터만 조회하므로 불필요, 넣으면 N+1)
- 날짜 직렬화: `createdAt`/`updatedAt`은 ISO-8601 UTC, `dueDate`는 `yyyy-MM-dd`
- DTO: `TodoCreateRequest`, `TodoUpdateRequest`, `TodoResponse`
- **개발용 시드 스크립트** — 용도가 둘이므로 파일을 나눈다
  - `db/seed-dev.sql` — 테스트 계정 1개 + Todo **100건** (우선순위·완료·마감일 혼합). 페이지네이션·필터·정렬을 **눈으로** 확인하는 용도
  - `db/seed-perf.sql` — 같은 계정에 Todo **10,000건**. 성능 DoD 측정 전용. 제목에 검색 대상 키워드가 골고루 섞이도록 생성한다
- **통합 테스트 4~7번 작성**

**DoD**

- [x] 목록 API가 `{success, data:{content, page, ...}, error}` 형태로 응답
- [x] `PUT` 저장이 완료 상태를 덮어쓰지 않음
- [x] `toggle`을 같은 값으로 두 번 호출해도 결과가 동일함(멱등)
- [x] `completed` 미지정 시 전체 반환, `true`/`false` 시 필터 적용
- [x] 영문 대소문자를 섞어 검색해도 결과가 나옴
- [x] `?sort=foo,desc` 같은 잘못된 정렬 값에도 500이 나지 않음
- [x] 삭제 시 `deleted_at` 기록, 목록에서 제외
- [x] 타 사용자 Todo 접근 시 404
- [x] 제목 미입력·200자 초과 시 400 + 필드 메시지
- [x] 본문 50,000자 초과 시 400
- [x] `<script>` 포함 본문 저장 시 태그 제거, `a` 태그에 `rel` 주입 확인
- [x] **키워드 검색 포함** 목록 조회가 **워밍업 후 3회 측정 중앙값 500ms 이내** (로컬, 시드 **10,000건** 기준) — 실측 37.3/38.8/40.4ms, 중앙값 **38.8ms**
  > ⚠️ 시드 100건으로는 이 지표가 의미가 없다. 인덱스가 없어도 100행은 1ms 미만이라 **항상 통과한다.** `CLAUDE.md` 4장이 지목한 유일한 성능 위험(`LOWER(title) LIKE '%키워드%'`의 인덱스 미사용)을 검출하려면 데이터가 충분해야 하고, 측정 대상도 검색 경로여야 한다. 또 첫 요청은 JVM 콜드 스타트라 DB가 아니라 워밍업 상태를 재는 셈이 되므로 워밍업 후에 측정한다.
- [x] 날짜가 배열이 아닌 ISO 문자열로 직렬화됨
- [x] 목록 조회 시 user 조회 쿼리가 추가로 발생하지 않음 — 100건 목록 조회 시 `todos` SELECT 1회 + `count(*)` 1회만 발생(인증 필터의 사용자 조회 1회는 요청당 고정이며 Todo 건수와 무관)
- [x] Swagger에서 전체 API 확인 가능
- [x] **Todo 통합 테스트 4건 통과**

> ### Phase 4 구현 기록 (2026-08-31)
>
> Todo 엔티티·Repository 기본 메서드는 Phase 2에서 완성되어 있었으나, `TodoController`·`TodoService`·`HtmlSanitizer`·DTO는 전혀 없어 순수 신규 구현으로 진행했다.
>
> - **검색 성능**: `LOWER(title)` 함수 기반 인덱스(`idx_todos_title_lower`)를 `db/add-title-index.sql`로 수동 생성했다(이 프로젝트에 Flyway/Liquibase가 없어 JPA `ddl-auto`로는 함수 표현식 인덱스를 만들 수 없음). 10,000건 시드 기준 키워드 검색 중앙값 **38.8ms**로 500ms DoD를 여유롭게 통과했다. 다만 `EXPLAIN ANALYZE` 확인 결과 이 규모(선택도 20%)에서는 PostgreSQL 플래너가 인덱스 대신 Seq Scan을 선택한다 — 실행시간(11.8ms)이 이미 충분히 빠르기 때문이며, 데이터가 더 커지거나 검색 조건의 선택도가 낮아지면 플래너가 인덱스를 선택하게 된다. 성능 목표는 달성했고 인덱스도 정확히 존재하지만, 이번 규모에서 실행계획에 나타나지는 않는다는 사실을 기록해 둔다.
> - **버그 발견 및 수정**: `keyword` 파라미터가 없을 때(`GET /api/todos`, completed만 지정) `500 lower(bytea) 이름의 함수가 없음`이 발생했다. JPQL의 `:keyword`가 null일 때 PostgreSQL이 파라미터 타입을 추론하지 못해 `bytea`로 잘못 캐스팅하는 문제였다. `TodoRepository.search`의 JPQL에 `cast(:keyword as string)`을 명시해 해결했다.
> - **HTML 정화**: Jsoup `Safelist.none()`에서 시작해 Phase 6 Tiptap 최종 설정과 대응하는 태그(`p,h2,h3,strong,em,ul,ol,li,blockquote,pre,code,br,a`)만 허용하고, `a[href]`에 `rel="nofollow noopener noreferrer"`를 강제 주입하며 `http`/`https`/`mailto` 외 스킴(예: `javascript:`)을 차단한다.

---

## Phase 5 — 구글 OAuth2 + OAuth 테스트

**저장소**: `todo-backend` · **관련 요구사항**: AUTH-05, AUTH-09

**작업**

- Google Cloud Console에서 OAuth 클라이언트 생성, 리다이렉트 URI 등록 (로컬 + 운영 둘 다)
- `spring.security.oauth2.client` 설정
- `CustomOAuth2UserService`: 신규 가입 / 기존 조회 / **충돌 거부** 분기
- **nickname 결정**: 구글 `name` → 없으면 이메일 `@` 앞부분 → 50자 초과 시 절삭
- `OAuth2SuccessHandler`: JWT 발급 후 `{FRONTEND_URL}/oauth/callback?token=` **302 리다이렉트**
- `OAuth2FailureHandler`: 충돌 시 `{FRONTEND_URL}/login?error=email_conflict` **302 리다이렉트** (JSON 에러 응답을 반환하지 않는다)
- **테스트 8번 작성 — 통합 테스트가 아니라 `CustomOAuth2UserService` 단위 테스트로 작성한다.** OAuth2 흐름은 실제 구글 서버와 통신하므로 MockMvc로 끝까지 검증할 수 없다 (`CLAUDE.md` 14장)

**계정 충돌 정책 (확정)**
동일 이메일의 로컬 계정이 있으면 **거부한다.** 자동 연동하지 않고, 별도 계정도 만들지 않는다. 상세는 `CLAUDE.md` 5장.

**DoD**

- [x] 구글 로그인 후 JWT를 담은 302 리다이렉트 발생 — 실측: `/oauth2/authorization/google` → 구글 동의 → `http://localhost:3000/oauth/callback?token=eyJ...` 리다이렉트 확인
- [x] 신규 사용자가 `provider=GOOGLE`, nickname이 채워진 상태로 저장됨 — 발급된 토큰으로 `/api/auth/me` 호출 시 `{"nickname":"정석","email":"ojs933327@gmail.com"}` 확인, 서버 로그에 `users` INSERT 1건만 발생(재로그인 시엔 발생하지 않음 — 아래 항목)
- [x] 재로그인 시 중복 계정이 생기지 않음 — 같은 구글 계정으로 재로그인한 새 토큰의 `sub` 클레임이 최초 로그인과 동일(`6`), 서버 로그에 `users` SELECT만 발생하고 INSERT 없음(`refresh_tokens`에만 새 행 발급)
- [x] 동일 이메일 로컬 계정 존재 시 `error=email_conflict`로 302 리다이렉트 — **단위 테스트로 검증.** 브라우저 실측에는 로컬 계정과 동일한 이메일의 별도 Google 테스트 사용자가 필요해(현재 테스트 계정은 이미 GOOGLE로 가입됨) 이번 라운드에서는 실행하지 않았고, `CustomOAuth2UserServiceTest.동일_이메일_로컬계정이_있으면_거부한다`가 실제 `UserRepository` 조회 결과를 목으로 구성해 `OAuth2AuthenticationException(email_conflict)` 발생을 확인함
- [x] **OAuth 서비스 단위 테스트 1건 통과** — 3건(`신규_이메일이면_구글_계정을_생성한다` / `기존_구글계정_재로그인시_새로_생성하지_않는다` / `동일_이메일_로컬계정이_있으면_거부한다`) 모두 통과, 전체 회귀 테스트 19건도 통과

---

## Phase 6 — 프론트 스캐폴딩

**저장소**: `todo-frontend` · **관련 요구사항**: UX-01·02·04·05·07 (공용 컴포넌트·토큰 수준) · **선행 조건**: Phase 0만 필요하다. **백엔드 Phase 1~5와 병렬로 진행할 수 있다** (이 Phase는 실서버를 호출하지 않는다)

**작업**

- **Node 20 이상** 확인 후 `create-next-app` (**Next.js 16.3.3 유지**, App Router, TypeScript)
- Tailwind CSS 4 설정 — `globals.css`의 `@theme`에 디자인 토큰 정의 (**v3 방식 금지**)
- **디자인 토큰은 라이트/다크 양쪽 정의 + `@media (prefers-color-scheme: dark)`** (`class` 전략 금지 — 토글이 없어 FOUC만 생김)
- shadcn/ui 초기화 (**npm 사용 시 `--legacy-peer-deps`**, 스타일 `new-york`), lucide-react 설치
- **`npm install motion`** (`framer-motion` 아님), sonner, date-fns, DOMPurify 설치
- **Tiptap 설치**: `@tiptap/react`, `@tiptap/starter-kit` **두 개만.** `@tiptap/extension-link`는 **설치하지 않는다** — v3 StarterKit에 `Link`가 포함되어 있어 중복 등록이 된다 (`CLAUDE.md` 8장)
- **Pretendard 폰트** — Google Fonts에 없으므로 `.woff2` 파일을 `src/app/fonts/`에 넣고 **`next/font/local`**로 로드 (`next/font/google` 사용 불가)
- React Query Provider, **쿼리 키 규약 상수화** (아래 「Phase 6 확정 값」), `apiClient` (토큰 주입 + **`ApiResponse` 언래핑** + 에러 정규화 + 401 처리)
- **`lib/errorMessages.ts` — 백엔드 `ErrorCode` enum을 코드로 옮긴다.** `error.code` **7종**(`INVALID_INPUT` / `EMAIL_DUPLICATED` / `UNAUTHORIZED` / `RESET_TOKEN_INVALID` / `FORBIDDEN` / `NOT_FOUND` / `INTERNAL_ERROR`)과 **네트워크 실패**를 화면 문구로 변환하는 단일 함수를 둔다
  > ⚠️ **`TODO_NOT_FOUND`라는 코드는 백엔드에 존재하지 않는다.** 실제 이름은 `NOT_FOUND`다. 정본은 `todo-backend/.../exception/ErrorCode.java`이며, 이 enum이 이미 한국어 `defaultMessage`를 갖고 있다(`PRD.md`에 「에러 문구 매핑」 표는 없다 — 이 참조는 오류였다).
  > ⚠️ 화면마다 문구를 직접 쓰면 Phase 7·8에서 서로 다른 문구가 생겨 매핑 표가 사문화된다. `apiClient`가 던지는 에러를 이 함수 하나로만 문구화한다. 네트워크 실패는 `error.code`가 없으므로 **정규화 단계에서 별도 구분자를 남겨야** 한다.
- **`lib/validation.ts` — 아래 「Phase 6 확정 값」 입력값 제약을 코드로 옮긴다.** (`CLAUDE.md` 4장은 「코딩 컨벤션」이고 제약 표는 없다 — 이 참조는 오류였다.) 폼 라이브러리를 쓰지 않기로 확정했으므로(`CLAUDE.md` 3장) 검증이 화면에 흩어지기 쉽다. 이메일 형식·닉네임 1~50자·제목 200자·본문 50,000자와 **비밀번호 6자 이상 + UTF-8 72바이트 이하**를 한곳에 둔다
  > ⚠️ **비밀번호 상한은 문자 수가 아니라 바이트다.** `maxLength={64}` 같은 문자 수 제한만 걸면 한글 25자(=75바이트)가 통과해 서버 BCrypt 단계에서 터진다. `new TextEncoder().encode(v).length`로 센다.
- `lib/sanitize.ts` (DOMPurify 래퍼) — **`ALLOWED_TAGS`·`ALLOWED_ATTR`을 명시한다.** 기본값은 Jsoup 화이트리스트보다 넓고, `ALLOWED_ATTR`에서 `rel`·`target`을 빠뜨리면 서버가 주입한 tabnabbing 방어가 렌더 단계에서 지워진다. 허용 태그의 정본은 `todo-backend/.../service/HtmlSanitizer.java`의 `Safelist`다
- 공용 컴포넌트: `Pagination`, `EmptyState`, `ErrorState`, `Skeleton`
  - **`ErrorState`는 `onRetry`를 필수 prop으로 받고 재시도 버튼을 항상 렌더한다** (`UX-04`). 선택 prop으로 두면 호출부에서 빠뜨려도 타입 검사가 통과한다
- 루트 레이아웃 + Provider 분리 (루트는 서버 컴포넌트, Provider는 클라이언트 컴포넌트)
- 공통 헤더 **껍데기만** 만든다 (닉네임·로그아웃 자리는 비워둠 — `useAuth`가 없는 시점이므로 Phase 7에서 연결)
- 타입 정의 (`src/types/`) — 백엔드 DTO와 이름 일치, `ApiResponse<T>` / `PageResponse<T>` 포함
- `.env.example`, 저장소용 `CLAUDE.md` (상단에 `@../CLAUDE.md` 임포트)
- **`public/static` 경로를 만들지 않는다** (Amplify 예약 경로)

**DoD**

- [x] **`package.json`의 `next`와 `eslint-config-next` 버전이 서로 같음 (둘 다 16.3.3)** — 둘 중 하나만 확인하면 놓친다. 15.x 다운그레이드는 2026-09-01에 **철회**됐다(위 정합성 점검 표 참조)
- [x] **소스가 `src/` 아래에 있음** (`src/app/`, `src/components/`, `src/lib/`, `src/types/`)
- [x] **`package.json`에 `@tiptap/extension-link`가 없음** (v3 StarterKit 내장)
- [x] Node 20 이상에서 빌드됨
- [x] `npm run build` 성공, 출력 디렉토리가 `.next`
- [x] `package.json`에 `motion`이 있고 `framer-motion`이 없음
- [x] 디자인 토큰이 OS 다크 설정에 따라 전환됨 (`class` 조작 없이 CSS만으로)
- [x] 페이지 `page.tsx`에 `"use client"`가 붙어 있음
- [x] **`components.json`의 `style`이 `new-york`임** (현재 `radix-nova`)
- [x] **`globals.css`에 `.dark` 클래스 셀렉터나 `@custom-variant dark (&:is(.dark *))`가 없고, 다크 토큰이 `@media (prefers-color-scheme: dark)` 안에 정의되어 있음** (현재 shadcn 기본값이 `class` 전략이라 반드시 걷어내야 한다)
- [x] 디자인 토큰 값이 아래 「Phase 6 확정 값」 팔레트와 일치함 (배경 `#FAFAFA`/`#0A0A0A`, 브랜드색 `#4F46E5`, 우선순위 3색) — shadcn 기본 neutral이 남아 있지 않음
  > ⚠️ `CLAUDE.md`에 8장은 없다(6장까지). 팔레트 정본은 `src/app/globals.css`의 `@theme` 토큰이며(`docs/guides/styling-guide.md` 86줄 규정), 확정 값은 아래 표에 기록한다.
  > ⚠️ **브랜드색 `#4F46E5`를 shadcn `--accent` 토큰에 넣지 않는다.** shadcn 규약에서 `accent`는 호버 시 깔리는 옅은 표면색이다. 브랜드색은 `--primary`·`--ring`에 넣는다.
- [x] Pagination 컴포넌트 단독 동작 확인 (더미 데이터). **페이지 수 1 이하일 때 아무것도 렌더하지 않음**
- [x] `ErrorState`가 재시도 버튼과 함께 렌더되고, 버튼 클릭이 `onRetry`를 호출함
- [x] `apiClient`가 `data` 언래핑과 401 처리를 수행함
- [x] **`apiClient`가 던진 에러를 `lib/errorMessages.ts`에 넣으면 백엔드 `ErrorCode`의 문구가 그대로 나옴** (네트워크 실패 케이스 포함 — 서버를 내리고 확인)

### Phase 6 재검증 / 발견 기록 (2026-09-01)

DoD 15개 항목을 전부 실제 명령으로 확인했다. 확인 방법과, 그 과정에서 문서에 없던 문제를 잡아낸 내역이다.

**검증에 쓴 명령·결과**

| 항목 | 확인 방법 | 결과 |
| --- | --- | --- |
| 빌드·정적검사 | `npm run check`(type-check+lint+format:check), `npm run build` | 둘 다 종료코드 0, `.next` 생성 |
| Node | `node -v` | v24.18.0 (20 이상) |
| 버전·패키지 | `package.json` 직접 조회 | `next`/`eslint-config-next` 모두 16.3.3, `motion` 있음, `framer-motion`·`@tiptap/extension-link`·`react-hook-form`·`zod` 없음 |
| 다크 전환 | Playwright `emulateMedia({colorScheme})`로 라이트↔다크 전환 후 computed style 측정 | 배경 `rgb(250,250,250)`↔`rgb(10,10,10)`, `--primary` `#4f46e5`↔`#818cf8`. `html`의 class는 두 상태에서 동일하고 `.dark`도 없다 — **CSS만으로 전환됨** |
| Pretendard | computed `font-family` | `pretendard` (fallback 아님) |
| Pagination | `totalPages=1`과 `5`를 함께 렌더 | 1일 때 컨테이너 `innerHTML`이 빈 문자열, `nav` 0개 |
| ErrorState | 재시도 버튼 2회 클릭 | 카운터 `0` → `2` (`onRetry` 실제 호출) |
| 터치 타겟 | 버튼 `getBoundingClientRect()` | 44×44px |
| `apiClient`·`errorMessages` | 백엔드를 **8081**에 띄우고 Node로 모듈을 직접 임포트해 실행 | 아래 참조 |

**`apiClient` 실측 (살아있는 백엔드 대상)**

- 인증 없이 `GET /api/todos` → 401 → refresh 1회 시도 → 실패 → `{kind:'api', code:'UNAUTHORIZED', status:401}`로 정규화
- 회원가입 → 반환값이 `{accessToken}` 하나뿐. **`refreshToken`이 본문에 없다**(`PRD.md` NF-26 준수)
- 목록 조회 → 반환값 키가 `content,page,size,totalElements,totalPages`이고 `success`가 없다 — **`ApiResponse` 봉투가 벗겨졌다**
- 없는 리소스 → `NOT_FOUND` → "요청한 리소스를 찾을 수 없습니다."
- 잘못된 입력 → `INVALID_INPUT` → "입력값이 올바르지 않습니다."
- **죽은 포트 → `{kind:'network'}` → "네트워크 연결을 확인해 주세요. 잠시 후 다시 시도해 주세요."**

**문서에 없던 문제 6건 (실측으로 발견)**

1. **`TODO_NOT_FOUND`는 존재하지 않는 코드다.** 백엔드 `ErrorCode` enum의 실제 값은 7종이고 이름은 `NOT_FOUND`다. ROADMAP대로 만들었으면 프론트가 서버 코드와 매칭되지 않아 전부 fallback 문구로 떨어졌을 것이다. 관련 표기를 모두 정정했다.
2. **날짜가 9시간 어긋난다.** `TodoResponse`의 `createdAt` 등이 `LocalDateTime`이고 Jackson 날짜 설정이 없어 `Z` 없이 직렬화된다. `lib/datetime.ts`의 `parseServerDateTime`이 흡수한다. `"2026-09-01T12:00:00.123456"` → `2026-09-01T12:00:00.123Z` → KST 표시 21:00으로 실측 확인했다.
3. **`prettier-plugin-tailwindcss`도 `src/` 이동에 걸린다.** `.prettierrc.json`의 `tailwindStylesheet`가 옛 `app/globals.css`를 가리켜 `npm run format`이 26개 파일에서 전부 실패했다. `tsconfig`·`components.json`과 **함께 고쳐야 하는 네 번째 설정 위치**다.
4. **포커스 링이 그려지지 않았다.** shadcn 컴포넌트의 `outline-none`이 `--tw-outline-style: none`을 고정시키고, `@layer base`의 `@apply outline-2`가 그 변수를 참조하므로 `outline-style`이 계속 `none`이었다. `:focus-visible` 규칙을 **`@layer` 밖**으로 빼서 해결했다(레이어 없는 선언이 모든 레이어보다 우선). 수정 후 `solid 3px`로 실제 렌더됨을 확인했다.
5. **브랜드색을 `--accent`에 넣으면 안 된다.** shadcn 규약에서 `accent`는 호버 표면색이다. `--primary`·`--ring`에 넣고 `--accent`는 옅은 인디고로 따로 뒀다.
6. **`AGENTS.md`는 삭제해도 되돌아온다.** `next dev`가 재생성한다(파일 본문에 명시). Next 16 유지이므로 생성물로 인정해 커밋하고, `todo-frontend/CLAUDE.md`를 `@AGENTS.md` → `@../CLAUDE.md`로 교체했다.

**남은 사항**

- **Pretendard Variable이 2,009KB다.** dynamic subset은 90여 개 파일에 `unicode-range` CSS가 필요해 `next/font/local`과 맞지 않아 전체 파일을 썼다. **Phase 9 최적화 항목**으로 남긴다.
- **로컬 8080을 다른 앱이 점유 중이다.** `C:\SpringBootProject\zulu17`의 별개 Spring Boot 프로세스다. 이번 검증은 백엔드를 8081에 띄워 진행했다. Phase 7 개발 전에 정리가 필요하다.
- **백엔드에 로그아웃 엔드포인트가 없다.** `controller/`·`SecurityConfig.java` 어디에도 `logout`이 없는데 `CLAUDE.md` 5장은 "클라이언트 토큰 삭제만으로 끝내지 않는다"를 절대 규칙으로 둔다. Phase 7이 헤더 로그아웃을 연결하는 단계이므로 **그 전에 결론이 필요하다.**

### Phase 6 확정 값 (2026-09-01)

`CLAUDE.md`·`PRD.md`에 해당 표가 존재하지 않아 이 Phase에서 확정했다. **이 표가 정본이며 코드는 이것을 옮긴 것이다.**

#### 팔레트

| 토큰 | 라이트 | 다크 | 비고 |
| --- | --- | --- | --- |
| `--background` | `#FAFAFA` | `#0A0A0A` | |
| `--foreground` | `#0A0A0A` | `#FAFAFA` | |
| `--primary` / `--ring` | `#4F46E5` | `#818CF8` | **브랜드색.** shadcn `--accent`가 아니다 |
| `--priority-low` | `#047857` | `#34D399` | 대비 5.25:1 / 10.30:1 |
| `--priority-medium` | `#B45309` | `#FBBF24` | 대비 4.81:1 / 11.86:1 |
| `--priority-high` | `#DC2626` | `#F87171` | 대비 4.63:1 / 7.16:1 |

대비비는 각 테마 배경(`#FAFAFA`/`#0A0A0A`) 기준 WCAG 상대휘도로 계산했고 전부 **4.5:1 이상**이다(`PRD.md` NF-23).
우선순위는 색만으로 구분하지 않고 **뱃지 텍스트 라벨을 항상 동반**한다(`PRD.md` NF-24).

#### React Query 쿼리 키 규약

```ts
authKeys.all          = ['auth'] as const
authKeys.me           = ['auth', 'me'] as const
todoKeys.all          = ['todos'] as const
todoKeys.lists        = ['todos', 'list'] as const
todoKeys.list(params) = ['todos', 'list', params] as const
todoKeys.details      = ['todos', 'detail'] as const
todoKeys.detail(id)   = ['todos', 'detail', id] as const
```

접두사가 계층을 이루므로 `todoKeys.all`로 무효화하면 목록·상세가 모두 걸린다. 생성/수정/삭제 후에는 `todoKeys.lists`를, 단건 수정 후에는 `todoKeys.detail(id)`를 무효화한다.

#### 입력값 제약

| 대상 | 제약 | 근거 |
| --- | --- | --- |
| 이메일 | 형식 검증, 소문자 정규화는 서버 담당 | `PRD.md` NF-12 |
| 비밀번호 | **6자 이상 AND UTF-8 72바이트 이하** | BCrypt 72바이트 한계 |
| 닉네임 | 1~50자 | `SignupRequest` `@Size(min=1,max=50)` |
| 제목 | 1~200자 | `TodoCreateRequest` `@Size(max=200)` |
| 본문 | 50,000자 이하 | `TodoCreateRequest` `@Size(max=50000)` |

> ⚠️ 비밀번호 상한은 **문자 수가 아니라 바이트**다. `maxLength={64}` 같은 문자 수 제한만 걸면 한글 25자(=75바이트)가 통과해 서버 BCrypt 단계에서 터진다. `new TextEncoder().encode(v).length`로 센다.

#### 날짜 직렬화 (실측 발견 — 문서에 없던 함정)

백엔드 `TodoResponse`의 `createdAt`·`updatedAt`·`completedAt`은 `LocalDateTime`이고 `application.yml`에 Jackson 날짜 설정이 없어 **`Z` 접미사 없이** `"2026-09-01T12:00:00.123456"` 형태로 직렬화된다. JS `new Date()`는 오프셋 없는 ISO 문자열을 **로컬 시각으로 해석**하므로 KST 브라우저에서 9시간 어긋난다. 백엔드를 고치지 않고 프론트 `lib/datetime.ts`의 `parseServerDateTime` 단일 진입점에서 `Z`를 붙여 흡수한다. `dueDate`는 `LocalDate`(`"2026-09-01"`)이므로 **`Z`를 붙이면 안 되며** 별도 함수로 분리한다.

> 버전·설치 방법은 `CLAUDE.md` 3장에서 모두 확정됐다. 이 Phase에서 재조사하지 않는다.

---

## Phase 7 — 인증 화면

**저장소**: `todo-frontend` · **관련 요구사항**: AUTH-01~09, UX-01, UX-06 · **선행 조건**: 백엔드 **Phase 3 완료**(가입·로그인·`/auth/me`), 구글 로그인 DoD는 **Phase 5 완료** 필요. 백엔드를 로컬에서 띄운 상태로 진행한다

**작업**

- `/login`, `/signup`, `/oauth/callback`
- **`/oauth/callback`은 `useSearchParams`를 쓰므로 `<Suspense>`로 감싼다** (없으면 `npm run build` 실패)
- **`/oauth/callback` 세부** (`PRD.md` 5.4)
  - 토큰 저장 → **URL에서 토큰 제거**(히스토리·공유 링크에 남지 않도록) → `/todos` 이동
  - **`token` 파라미터가 없거나 빈 문자열이면 `/login`으로 보낸다**
  - 처리 중에는 스켈레톤만 보여주고 **사용자가 조작할 요소를 두지 않는다**
  - **공통 헤더를 두지 않는다** (인증 처리 중 화면이라 닉네임을 알 수 없다)
- **`/todos` 플레이스홀더 페이지 생성** — 라우트 보호를 검증하려면 대상 페이지가 존재해야 한다 (내용은 Phase 8)
- Phase 6에서 비워둔 **헤더의 닉네임·로그아웃을 `useAuth`에 연결**
  - **이메일은 화면에 표시하지 않는다.** `/auth/me` 응답에는 들어오지만 헤더에는 닉네임만 노출한다 (`AUTH-08`)
- `useAuth` 훅 (로그인 / **로그아웃: 토큰 삭제 + 캐시 초기화** / 현재 사용자)
- **라우트 보호는 `(main)` 클라이언트 레이아웃에서 처리한다. `middleware.ts`를 만들지 않는다** (localStorage는 middleware에서 읽을 수 없음 — `CLAUDE.md` 9장)
  - **인증 판정이 끝나기 전에는 스켈레톤을 보여준다** (`UX-01`, `PRD.md` 5.1)
- 401 응답 시 자동 로그아웃 처리
- `?error=email_conflict` 안내 문구 표시
- **`/signup` 실시간 검증** (`PRD.md` 5.3) — 이메일 형식, **비밀번호 6자 이상 + UTF-8 72바이트 이하**, 닉네임 1~50자. 안내 문구에 **한글 1자 = 3바이트**임을 밝힌다
- **에러 문구는 Phase 6의 `lib/errorMessages.ts`만 사용한다** (정본은 백엔드 `ErrorCode` enum)
  - `INVALID_INPUT` → 서버가 준 필드별 메시지를 해당 입력 아래 인라인
  - `UNAUTHORIZED`(로그인 시) → "이메일 또는 비밀번호가 올바르지 않습니다."를 폼 상단 인라인
  - `UNAUTHORIZED`(그 외) → 문구 없이 `/login` 이동
  - `EMAIL_DUPLICATED` → "이미 사용 중인 이메일입니다."를 이메일 입력 아래 인라인
  - 네트워크 실패 → "연결에 실패했습니다." + 재시도 버튼

**DoD**

- [ ] 이메일 가입·로그인 정상 동작
- [ ] **가입 화면에서 한글 25자 비밀번호를 입력하면 제출 전에 바이트 초과 안내가 인라인으로 뜸** (서버 400에만 의존하지 않음 — `AUTH-02`)
- [ ] **중복 이메일 가입 시 "이미 사용 중인 이메일입니다."가 이메일 입력 아래 인라인으로 뜸** (`AUTH-03`, `PRD.md` 5.1)
- [ ] **미가입 이메일과 비밀번호 오류의 화면 문구가 동일함** ("이메일 또는 비밀번호가 올바르지 않습니다.") — 계정 존재 여부가 드러나지 않음
- [ ] **백엔드를 내린 채 로그인을 시도하면 "연결에 실패했습니다." + 재시도 버튼이 나옴** (네트워크 실패 매핑 — `UX-04`)
- [ ] 구글 로그인 → 콜백 → `/todos` 이동, URL에서 토큰 제거됨
- [ ] **`/oauth/callback`에 `?token=` 없이 직접 접근하면 `/login`으로 이동함** (`PRD.md` 5.4)
- [ ] **`/oauth/callback` 화면에 공통 헤더와 조작 가능한 요소가 없음**
- [ ] **헤더에 닉네임이 보이고 이메일은 어디에도 렌더되지 않음** (DevTools에서 DOM 검색 — `AUTH-08`)
- [ ] 계정 충돌 시 안내 문구 노출
- [ ] 로그아웃 시 토큰·캐시 모두 제거되고 `/login`으로 이동
- [ ] 새로고침해도 로그인 상태 유지
- [ ] 미인증 상태로 `/todos` 접근 시 로그인으로 이동
- [ ] **만료된 토큰을 localStorage에 직접 넣고 `/todos`에 접근했을 때, 보호 화면이 한 프레임도 노출되지 않고 곧바로 `/login`으로 이동**
  > `useAuth`가 토큰 존재 여부만 보면 만료 토큰이 판정을 통과해, 401 왕복 동안 보호 화면이 노출된다. `exp`를 디코드해야 한다 (`CLAUDE.md` 9장). 검증용 만료 토큰은 `JWT_EXPIRATION`을 일시적으로 낮춰 발급받으면 된다
- [ ] `middleware.ts` 파일이 존재하지 않음
- [ ] `npm run build` 성공 (`useSearchParams` Suspense 경계 확인)
- [ ] **모든 입력에 label 연결, Tab·Enter만으로 가입·로그인 완주 가능**

---

## Phase 8 — Todo 화면

**저장소**: `todo-frontend` · **관련 요구사항**: TODO-01~10, 12~16, UX-01~06 · **선행 조건**: 백엔드 **Phase 4 완료**(Todo API 6종 + 시드). `db/seed-dev.sql`을 로컬에 적용한 상태로 진행해야 페이지네이션·필터 DoD를 눈으로 확인할 수 있다

**작업**

- `/todos`: 목록, 검색, 완료 필터, 페이지네이션
- **검색어·필터·페이지는 URL 쿼리로 관리** (`?page=2&completed=false&keyword=...`)하고, 페이지 전체를 `<Suspense>`로 감싼다
- `useTodos` 훅 (React Query)
- **`TodoForm` 공용 컴포넌트** → `/todos/new`와 `/todos/[id]`가 재사용 (진입 즉시 편집 가능, 명시적 저장)
- **`TodoForm`에 완료 체크박스를 두지 않는다.** 완료는 목록에서만 변경
- **Tiptap 통합 — StarterKit을 기본값으로 쓰지 않는다.** `heading.levels [2,3]`, `strike: false`, `horizontalRule: false`, **`underline: false`**로 설정하고 **`link`는 StarterKit 내장 옵션으로 설정**한다 (v3에서 Link·Underline이 StarterKit에 포함됨 — `CLAUDE.md` 8장)
- **`editor.commands.setContent()` 호출 직전에 `lib/sanitize.ts`로 DOMPurify 정화** — 이 앱에는 `dangerouslySetInnerHTML`이 없으므로 여기가 유일한 렌더 방어 지점이다 (`CLAUDE.md` 6장)
- 우선순위 뱃지, 마감일 표시 (date-fns 포맷)
- **완료 항목은 제목에 취소선 + 흐린 색상**을 적용한다 (`PRD.md` 5.5)
- **`/todos/[id]` 저장 실패 처리** — 폼 내용을 유지한 채 에러를 표시한다. 입력을 날리거나 목록으로 튕기지 않는다 (`TODO-13`, `UX-04`, `PRD.md` 5.6)
  > ⚠️ `TODO-13`은 Phase 9의 토글·삭제 롤백만으로 충족되지 않는다. **저장(PUT) 실패 경로는 낙관적 업데이트를 쓰지 않으므로 Phase 9가 손대지 않는다.** 이 Phase에서 별도로 처리한다.
- **`/todos/[id]` 삭제 성공 시 `/todos`로 이동**한다 (`TODO-12`, `PRD.md` 5.6)
- **에러 문구는 Phase 6의 `lib/errorMessages.ts`만 사용한다.** `NOT_FOUND`는 전체 화면 상태, `INTERNAL_ERROR`는 토스트 또는 에러 카드, 네트워크 실패는 재시도 버튼이 있는 에러 카드 (백엔드 `ErrorCode` enum)
- **이탈 확인 대화상자 — 3계층으로 구현** (`beforeunload` + 버튼 핸들러 + `popstate` 가드). App Router에 공식 차단 API가 없어 한 줄로 끝나지 않는다. **별도 공수 4~8시간을 잡는다** (`CLAUDE.md` 9장)
- **`dirty` 판정을 직접 구현한다.** 폼 라이브러리를 쓰지 않으므로 `formState.isDirty`가 없다. 제목·우선순위·마감일은 단순 비교로 끝나지만 **본문은 그렇지 않다** — Tiptap이 HTML을 자기 스키마로 정규화하므로 서버 원본과 `editor.getHTML()`을 직접 비교하면 사용자가 아무것도 고치지 않아도 dirty로 판정된다. **초기 스냅샷은 `setContent()` 직후의 `editor.getHTML()`로 잡는다**(정규화를 거친 값끼리 비교)
- **삭제 실패 시 페이지 이동까지 되돌린다** — 페이지 이동은 `onMutate`가 아니라 `onSuccess`에서 수행 (`CLAUDE.md` 9장)
- **경계 상황**: 마지막 항목 삭제로 페이지가 비면 이전 페이지로 이동, `/todos/[id]` 404 시 전용 화면 (`CLAUDE.md` 9장)
- 로딩 스켈레톤 / 빈 상태 / 검색 결과 없음 / 에러 상태

**DoD**

- [ ] **목록이 페이지당 정확히 10건씩 끊기고, 항목 순서가 생성일 내림차순임** (시드 데이터의 `created_at`과 대조 — `TODO-05`, `TODO-06`)
- [ ] 20건 이상에서 페이지네이션 정상
- [ ] **정렬 기준을 고르는 UI가 화면에 없음** (생성일 내림차순 고정 — `PRD.md` 1장 비목표)
- [ ] 검색·필터 상태가 URL에 반영되고 새로고침·뒤로가기에서 유지됨
- [ ] **완료 처리한 항목의 제목에 취소선과 흐린 색상이 적용됨** (`PRD.md` 5.5)
- [ ] `npm run build` 성공
- [ ] Tiptap 내용 저장 후 재조회 시 서식 유지 (툴바 항목 전부)
- [ ] 본문에 `# `, `~~취소선~~`, `---`를 입력해도 서식이 생성되지 않음 (저장 후 소실되는 입력이 없음)
- [ ] **본문에서 `Ctrl+U`를 눌러도 밑줄(`<u>`)이 생성되지 않음** (v3 StarterKit의 Underline이 꺼져 있는지 확인 — 켜져 있으면 저장 시 서식이 조용히 사라진다)
- [ ] 2페이지 이상에서 마지막 항목 삭제 시 이전 페이지로 이동
- [ ] **2페이지 마지막 항목 삭제가 실패했을 때, 사용자가 보고 있는 화면에서 롤백이 눈으로 확인됨** (페이지가 먼저 넘어가 버려 롤백이 안 보이면 실패)
- [ ] 타인 소유 id로 접근 시 "찾을 수 없습니다" 화면 표시 + **"목록으로 가기" 버튼이 있고, 자동 리다이렉트가 일어나지 않음** (`TODO-14`, `PRD.md` 5.6)
- [ ] `/todos/[id]` 로딩 중 **폼 형태 스켈레톤**이 보임 (`UX-01`)
- [ ] **백엔드를 내린 채 저장을 누르면 입력한 제목·본문이 그대로 남아 있고 에러가 표시됨** (`TODO-13`, `UX-04`)
- [ ] **`/todos/[id]`에서 삭제하면 `/todos`로 이동함** (`TODO-12`)
- [ ] **`setContent()` 직전에 `lib/sanitize.ts`가 호출됨** (본문에 `<script>`가 섞인 데이터를 DB에 직접 넣고 상세 화면 진입 시 실행되지 않음)
- [ ] `/todos/new`와 `/todos/[id]`가 `TodoForm`을 재사용
- [ ] 수정 화면에서 저장해도 완료 상태가 바뀌지 않음
- [ ] 변경 후 이탈 시 확인 대화상자 노출 (**새로고침 / 페이지 내 취소 버튼 / 브라우저 뒤로가기 3경로 모두**)
- [ ] **저장 직후에는 확인 대화상자가 뜨지 않음** (`dirty` 해제 확인)
- [ ] **본문이 있는 할 일을 열어 아무것도 고치지 않고 나갈 때 확인 대화상자가 뜨지 않음** (Tiptap 정규화로 dirty가 오판되지 않는지 — 서식이 섞인 본문으로 시험한다)
- [ ] 4가지 화면 상태 모두 눈으로 확인
- [ ] **에러 상태의 재시도 버튼을 눌렀을 때 실제로 재요청이 나가고, 서버를 다시 올리면 목록이 정상 렌더됨** (`UX-04` — 버튼이 보이기만 하고 동작하지 않는 경우를 걸러낸다)
- [ ] **빈 상태 문구("아직 할 일이 없어요")와 검색 결과 없음 문구가 서로 다름** (`UX-03`)
- [ ] 360px 화면에서 레이아웃 정상 (가로 스크롤 없음)
- [ ] **키보드만으로 할 일 생성·완료 토글·삭제 수행 가능**

---

## Phase 9 — 인터랙션 다듬기

**저장소**: `todo-frontend` · **관련 요구사항**: TODO-11~13 · **선행 조건**: Phase 8 완료(목록·상세 화면이 실제 API로 동작하는 상태). 백엔드 Phase 4의 `toggle` 멱등 동작이 전제다

> `TODO-13` 중 **저장(PUT) 실패 처리는 Phase 8에서 끝낸다.** 이 Phase가 다루는 것은 낙관적 업데이트를 쓰는 **토글·삭제**의 롤백뿐이다.

**작업**

- 완료 토글·삭제 낙관적 업데이트 (`onMutate` / `onError` 롤백 / `onSettled`)
- 토글은 **목표 상태를 그대로 서버에 전송** (서버 계산에 의존하지 않음)
- **연타 대비 — mutation 직렬화 또는 마지막 1회 invalidate.** React Query v5 mutation은 기본 병렬이라 목표 상태 전송(멱등)만으로는 요청 재정렬을 막지 못한다. `scope: { id: \`todo-toggle-${todoId}\` }`또는`onSettled`의 `isMutating() === 1` 가드를 적용한다 (`CLAUDE.md` 9장)
- Motion: 목록 등장(stagger), 삭제(`AnimatePresence`), 토글 스프링 — **import는 `motion/react`**
- 실패 시 토스트 알림(`sonner`)
- `prefers-reduced-motion` 대응

**DoD**

- [ ] 토글·삭제 시 대기 시간 없이 즉시 반영
- [ ] **체크박스를 빠르게 연타해도 새로고침 후 상태가 UI와 일치**
- [ ] **연타를 멈춘 뒤 최종 상태가 "마지막에 클릭한 값"과 일치하며, 잠시 후 반대 값으로 되돌아가지 않음**
  > 앞 항목만 보면 결함이 통과한다. `invalidateQueries`가 어떤 값으로든 수렴시키므로 "UI와 서버가 일치"는 항상 참이 된다. 문제는 **수렴한 값이 사용자 의도와 다를 수 있다는 것**이다
- [ ] 서버를 내린 상태에서 실패 → UI 롤백 + 알림 확인
- [ ] 애니메이션이 200ms 이내, 과하지 않음
- [ ] 애니메이션 관련 import가 모두 `motion/react`에서 이루어짐

---

## Phase 10 — 전체 검증

**저장소**: 전체 · **새 테스트를 작성하지 않는다.** 전체 통과와 아래 체크리스트만 확인한다.

**작업**

- 각 저장소 README 작성
- 전체 검증 체크리스트 수행

### 최종 검증 체크리스트 (완료 판정 정본)

**환경**

- [ ] `todolist_db`, `todolist_db_test`와 함께 PostgreSQL 실행
- [ ] `./mvnw spring-boot:run` 오류 없이 기동
- [ ] `./mvnw test` 전체 통과 (통합 테스트 8건 + Repository 단위 테스트)
- [ ] `npm run build` 성공
- [ ] Swagger UI에서 전체 API 확인
- [ ] 세 저장소의 브랜치가 `main`/`develop` 체계이고 `master`가 남아 있지 않음
- [ ] `CLAUDE.md`·`PRD.md`·`ROADMAP.md`가 서로를 참조하는 경로가 실제 파일 위치와 일치함

**인증**

- [ ] 회원가입 시 사용자 생성 및 JWT 반환
- [ ] 로그인 시 유효한 JWT 반환 (`sub`에 user id)
- [ ] 보호된 엔드포인트에 유효 토큰 필요
- [ ] 구글 소셜 로그인 정상 동작, nickname 채워짐
- [ ] 동일 이메일 로컬 계정 존재 시 구글 로그인 거부 및 안내
- [ ] 로그아웃 시 토큰·캐시 제거
- [ ] 헤더에 닉네임만 표시되고 이메일은 화면 어디에도 노출되지 않음 (`AUTH-08`)
- [ ] 만료 토큰으로 보호 화면 접근 시 화면 노출 없이 `/login`으로 이동 (`AUTH-07`)

**기능**

- [ ] Todo CRUD가 페이지네이션과 함께 작동
- [ ] 모든 응답이 `{success, data, error}` 포맷 (목록 포함)
- [ ] 완료 필터(미지정 시 전체)·제목 검색(대소문자 무시) 동작
- [ ] 수정 저장이 완료 상태를 덮어쓰지 않음
- [ ] 토글 연타 후에도 서버 상태와 UI 일치
- [ ] Soft Delete 시 `deleted_at` 갱신 및 목록 제외
- [ ] 타 사용자 리소스 접근 시 404
- [ ] Tiptap 저장/렌더링 정상, 우선순위·마감일 반영
- [ ] 낙관적 업데이트 및 실패 롤백 동작

**보안**

- [ ] `<script>` 포함 본문이 저장 시 정화됨
- [ ] 링크에 `rel="noopener noreferrer"` 주입됨 — **저장 시(Jsoup)뿐 아니라 렌더 후 DOM에서도 남아 있는지 확인.** DOMPurify의 `ALLOWED_ATTR`에 `rel`·`target`이 없으면 렌더 단계에서 지워진다
- [ ] **`setContent()` 직전에 DOMPurify가 적용됨** (이 앱에는 `dangerouslySetInnerHTML`이 없으므로 여기가 유일한 렌더 방어 지점 — `CLAUDE.md` 6장)
- [ ] **툴바 · Tiptap 확장 · Jsoup 화이트리스트 · DOMPurify `ALLOWED_TAGS` 네 곳의 태그 집합이 일치**
- [ ] 입력값 상한(비밀번호 **UTF-8 72바이트**, 제목 200자, 본문 50,000자) 검증 동작 — **한글 비밀번호로도 시험한다**
- [ ] 시크릿이 저장소에 커밋되지 않음

**UX**

- [ ] 로딩/빈 상태/검색 결과 없음/에러 상태 모두 확인 (**에러 상태의 재시도 버튼이 실제로 재요청을 보냄** — `UX-04`)
- [ ] `PRD.md` 5.1 에러 문구 매핑 6종이 화면에서 표 그대로 나옴 (`INVALID_INPUT` / `UNAUTHORIZED` 2경우 / `EMAIL_DUPLICATED` / `TODO_NOT_FOUND` / `INTERNAL_ERROR` / 네트워크 실패)
- [ ] 로그인 실패 문구가 계정 존재 여부를 구분하지 않음
- [ ] 360px ~ 1920px 반응형 정상
- [ ] **Chromium 계열 1종 + 사용 가능한 다른 엔진 1종에서 확인** (Mac은 Chrome+Safari, Windows는 Chrome+Edge/Firefox)
- [ ] OS 다크 설정에 따라 테마 전환
- [ ] 폼 label 연결 및 키보드 조작 가능

→ 전 항목 통과 시 세 저장소에 `v1.0.0` 태그

---

## Phase 11 — AWS 배포

**저장소**: 전체 · **Docker 사용하지 않음**

> ### ⚠️ 미해결 리스크 — Amplify의 Next.js 16 지원 (2026-09-01 확인)
>
> AWS 공식 문서는 이 시점에도 Amplify Hosting의 Next.js 지원 범위를 **12~15**로 명시하고 있고 **16은 목록에 없다.**
> 출처: [SSR supported features](https://docs.aws.amazon.com/amplify/latest/userguide/ssr-supported-features.html) · [Amplify support for Next.js](https://docs.aws.amazon.com/amplify/latest/userguide/ssr-amplify-support.html)
>
> 프로젝트는 2026-09-01에 **Next.js 16.3.3 유지**를 확정했다(실제 코드·`CLAUDE.md` 3장·`docs/guides` 5개 문서가 모두 16 기준이라 15 다운그레이드 비용이 더 컸다). 따라서 이 Phase 착수 시 **다음 중 하나를 먼저 결정해야 한다**:
>
> 1. 실제로 Amplify에 배포해 동작 여부를 확인한다 (미지원 목록이 곧 실패를 뜻하지는 않는다)
> 2. 배포 시점에 15.x로 다운그레이드한다
> 3. 호스팅을 바꾼다 (OpenNext + SST, Vercel, EC2 자체 호스팅 등)
>
> **이 결정 전에는 Phase 11을 시작하지 않는다.**

### 11-0. 사전 준비

- **도메인 확보** (미보유 시 구입). API용 서브도메인이 필요하다 — 예: 프론트 `todo.example.com`(Amplify), API `api.example.com`(EC2)
- Route 53 호스팅 영역 생성 또는 기존 DNS 제공자에서 레코드 관리 준비

### 11-1. 네트워크 & DB

- VPC 기본 구성, 퍼블릭/프라이빗 서브넷 확인
- **RDS는 프라이빗 서브넷에 배치**하고 퍼블릭 액세스를 비활성화한다
- RDS 보안그룹: 인바운드 5432를 **EC2 보안그룹에서만** 허용 (0.0.0.0/0 금지)
- RDS PostgreSQL 생성 후 `todolist_db` 데이터베이스 생성
- **스키마 적용** (`CLAUDE.md` 4장 절차)
  1. 로컬에서 DDL 스크립트 추출
  2. 검토 후 RDS에 1회 수동 적용
  3. 운영 프로파일을 `ddl-auto: validate`로 고정
  4. `db/schema.sql`을 저장소에 커밋
     > ⚠️ **RDS가 프라이빗 서브넷에 있으므로 로컬 PC에서 직접 접속할 수 없다.** 2번을 실행할 경로가 필요하다:
     > EC2에 `postgresql-client` 설치 → `scp`로 DDL 파일을 EC2에 전송 → **EC2에서 `psql -h <rds-endpoint> -U <user> -d todolist_db -f schema.sql`** 실행.
     > 이 경로를 준비하지 않으면 11-1 중반에 막힌다. EC2를 먼저 띄운 뒤 RDS 스키마를 적용하는 순서가 된다.

### 11-2. 백엔드 (EC2)

- EC2에 JDK 21 설치
- `./mvnw package`로 jar 생성 후 전송
- **systemd 서비스로 등록** (자동 재시작, 부팅 시 기동)
- 환경변수는 systemd `EnvironmentFile`로 주입 (`.env` 커밋 금지)
- EC2 보안그룹 인바운드: **80과 443을 공개**, 22는 본인 IP로 제한, 8080은 외부에 열지 않는다
  > ⚠️ **80을 닫으면 안 된다.** 11-3의 `80 → 443` 리다이렉트가 도달 불가능해지고, **certbot의 HTTP-01 챌린지도 실패해 인증서 발급 자체가 안 된다.** 80은 열되 nginx가 443으로 리다이렉트만 하도록 구성한다 (평문으로 서비스하지 않는다)

### 11-3. HTTPS (방식 확정: nginx + certbot)

개인 프로젝트 규모이므로 **EC2 한 대에 nginx 리버스 프록시 + Let's Encrypt**로 간다. ALB + ACM은 관리가 편하지만 상시 비용... (12KB 남음)

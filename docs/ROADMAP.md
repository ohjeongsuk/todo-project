# ROADMAP — Todo List 프로젝트

> **버전** 1.9 · **최종 수정** 2026-09-07
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
| 7     | 인증 화면                  | frontend | ✅   |
| 8     | Todo 화면                  | frontend | ✅   |
| 9     | 인터랙션 다듬기            | frontend | ✅   |
| 10    | 전체 검증                  | 전체     | ✅   |
| 11    | AWS 배포                   | 전체     | ⬜   |
| 12    | 로컬 이미지 첨부           | 전체     | ✅   |
| 13    | S3 전환                    | backend  | ✅   |
| 14    | 비밀번호 재설정            | 전체     | ✅   |

⬜ 대기 · 🟡 진행중 · ✅ 완료

> **Phase 0이 🟡인 이유 (2026-09-01):** DoD 9개 중 8개가 통과했고 **원격 푸시 하나만 남았다.** 세 저장소 모두 GitHub 원격이 연결돼 있으나 `main`·`develop`이 push되지 않았다. 푸시하면 Phase 0은 ✅가 된다.

> **Phase 10이 ✅인 이유 (2026-09-07 결정):** 최종 검증 체크리스트 37개 중 36개가 통과했다. 남은 1개("두 번째 렌더링 엔진 확인")는 이 PC에 Chromium 계열(Blink) 외의 브라우저 엔진이 전혀 없어(Firefox 미설치, Playwright도 Blink) 환경상 확인이 불가능하다. Firefox를 지금 설치하는 대신 **Phase 11 배포 완료 후 실기기·실브라우저로 확인하기로 결정**했다 — Phase 11 DoD에 항목을 추가했다. 이 결정으로 `v1.0.0` 태그는 아직 걸지 않았다(이연된 항목이 있는 상태에서의 태깅 여부는 별도 확인이 필요하다).

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
| AUTH-04 로그인·JWT 30분                  | 3 · 7                                                         | 3 · 7 · 10                                       |
| AUTH-05 구글 로그인                      | 5 · 7(`/oauth/callback`)                                      | 5 · 7 · 10                                       |
| AUTH-06 로그아웃                         | 7 (**서버 `POST /api/auth/logout` + 프론트**)                 | 7 · 10                                           |
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
| UX-07 다크 토큰 (`html[data-theme]` + 토글)  | 6 · 후속(2026-09-03)                                          | 6 · 10 · 후속                                    |
| IMG-01 이미지 첨부 (F-46)                | 12(백엔드 API · Tiptap 노드)                                  | 12                                               |
| IMG-02 업로드 제한 (F-47, NF-34)         | 12(타입 화이트리스트 · 크기 이중 방어)                        | 12(테스트 11번)                                  |
| IMG-03 첨부 소유권 (F-48)                | 12(쿼리 단계 강제, 404)                                       | 12(테스트 10번)                                  |
| IMG-04 본문 참조 방식 (F-49)             | 12(Jsoup · DOMPurify 양쪽 `img` 허용, `src` 미저장)           | 12(테스트 12번 · 수동 11·12·16)                  |
| IMG-05 스토리지 추상화 (F-50)            | 12(`StorageService` + 로컬 구현) · 13(S3 구현)                | 12 · 13(프론트 무변경 확인)                      |
| IMG-06 고아 파일 정리 (F-51)             | 12(`@Scheduled` 배치, 상태별 정책)                            | 12                                               |
| IMG-07 서명 URL (NF-32)                  | 12(jjwt 재사용, 용도·만료 서명)                               | 12(테스트 10번)                                  |
| IMG-08 경로 조작 방어 (NF-33)            | 12(`Path.normalize()` + base 하위 검증)                       | 12(단위 테스트)                                  |
| PWD-01 재설정 요청 (F-41)                | 14(`POST /api/auth/password/forgot`) · 14(`/forgot-password`) | 14                                               |
| PWD-02 재설정 링크 처리 (F-42)           | 14(`GET .../verify` + `POST .../reset`) · 14(`/reset-password`) | 14                                             |
| PWD-03 토큰 정책 1회용·30분 (F-43)       | 14(발급 시 기존 미사용 토큰 무효화)                           | 14                                               |
| PWD-04 재설정 후 전체 로그아웃 (F-44)    | 14(`revokeAllForUser` 재사용, 자동 로그인 안 함)              | 14                                               |
| PWD-05 소셜 전용 계정 차단 (F-45)        | 14(`hasPassword()` false면 발송 안 함, 응답 동일)             | 14                                               |
| PWD-06 rate limit (NF-30)                | 14(이메일·IP 10분 3회, 초과 시 429)                           | 14                                               |
| PWD-07 계정 존재 미노출 (NF-31)          | 14(없는 계정·소셜 계정·정상 계정 응답 동일)                   | 14                                               |

> **에러 문구 매핑**은 `PRD.md`에 표로 존재하지 않는다(`PRD.md` 5.1은 보안 비기능요구사항 NF-01~NF-31일 뿐이다). 정본은 백엔드 `ErrorCode` enum(7종)을 그대로 옮긴 `lib/errorMessages.ts`이며, Phase 6에서 단일화하고 Phase 7·8에서 화면별로 적용, Phase 10에서 7종 전부를 대조한다.
> `PRD.md` **5장** 비기능 요구사항의 검증 위치(응답 속도 4 · 체감 반응 9 · 브라우저 10 · 반응형 8·10 · 인증 보안 3·4 · 시크릿 관리 10·11 · XSS 4·8 · 문서화 4 · 접근성 7·8)는 위 표 및 각 Phase DoD와 일치한다.

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
- [x] 세 저장소 모두 첫 커밋 및 원격 푸시 완료 — **해소 (2026-09-07 재확인).** 세 저장소 모두 `origin/main`·`origin/develop`이 원격에 존재한다. 아래 2026-09-01 절차 기록은 그 시점 상태이며, 이후 실제로 푸시됐다.
  > **다만 로컬이 원격보다 앞선 상태가 반복된다.** 2026-09-07 기준 `develop`이 origin보다 `todo-project` 4·`todo-backend` 1·`todo-frontend` 1 커밋 앞서 있었고, `main`은 Phase 3~8 수준에 멈춰 `develop`을 한참 못 따라가고 있었다(Phase 12·13 결과물이 `main`에 하나도 없었다). `CLAUDE.md`가 정한 `feature → develop → main` 흐름이 유지되지 않은 것으로, Phase 14 착수 전에 세 저장소 모두 푸시·병합해 정리했다.

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
  > 푸시 전 확인: `gh` CLI가 설치돼 있지 않아 세 저장소가 public인지 private인지 확인하지 못했다. **public이면 푸시 즉시 코드가 공개된다.** `.env`·`application-local.properties`(당시 `application-local.yml`)는 무시 규칙에 걸려 있음을 종료코드로 확인했으나, 이미 커밋된 파일에 비밀값이 없는지는 푸시 직전에 한 번 더 본다.
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
  > ⚠️ **Phase 12에서 뒤집혔다.** 설정 파일은 다시 `.properties`로 전환됐고 `.yml`은 하나도 남아 있지 않다. 위 항목은 Phase 1 시점의 기록이다.
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
- `src/test/resources/application-test.properties` (todolist_db_test, `ddl-auto: create-drop`)
  > Phase 2 당시에는 `application-test.yml`이었다. Phase 12에서 `.properties`로 전환됐다.
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
  - **`permitAll` 경로에 Swagger(`/swagger-ui/**`, `/v3/api-docs/**`)를 반드시 포함**
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
- [x] `/api/auth/me` 응답에 `nickname`과 `email`이 모두 포함됨 (AUTH-08 — 화면 표시 여부는 Phase 7에서 통제)
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
- **HtmlSanitizer**: Jsoup Safelist, 허용 태그·`rel` 주입·스킴 제한 (`CLAUDE.md` 3장 절대 규칙 8)
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
> - **HTML 정화**: Jsoup `Safelist.none()`에서 시작해 Phase 6 Tiptap 최종 설정과 대응하는 태그(`p,strong,em,ul,ol,li,br,a`)만 허용하고, `a[href]`에 `rel="nofollow noopener noreferrer"`를 강제 주입하며 `http`/`https`/`mailto` 외 스킴(예: `javascript:`)을 차단한다.

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
- ~~**디자인 토큰은 라이트/다크 양쪽 정의 + `@media (prefers-color-scheme: dark)`** (`class` 전략 금지 — 토글이 없어 FOUC만 생김)~~
  → **번복됨 (2026-09-03).** 이 지침은 `PRD.md` F-33("라이트 기본 + 다크 **토글**. 선택은 유지되고 첫 로드 시 색이 번쩍이지 않는다")과 어긋난다. "토글이 없어서 미디어쿼리로 간다"는 근거가 순환 논리였다 — 토글이 없는 이유가 토글을 만들지 않았기 때문이다. 토글을 구현하면서 전환 신호를 `html[data-theme]`으로 바꿨다. 자세한 내용은 아래 「Phase 6 이후 변경」 참고
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
- [x] 디자인 토큰이 OS 다크 설정에 따라 전환됨 ~~(`class` 조작 없이 CSS만으로)~~ — 2026-09-03에 `html[data-theme]` 방식으로 바뀌었다. OS 연동은 "시스템"이 기본값이라 그대로 동작한다
- [x] 페이지 `page.tsx`에 `"use client"`가 붙어 있음
- [x] **`components.json`의 `style`이 `new-york`임** (현재 `radix-nova`)
- [x] ~~**`globals.css`에 `.dark` 클래스 셀렉터나 `@custom-variant dark (&:is(.dark *))`가 없고, 다크 토큰이 `@media (prefers-color-scheme: dark)` 안에 정의되어 있음**~~ (당시 판정 기준. shadcn 기본값인 `class` 전략을 걷어내는 것이 목적이었다)
  → **무효 (2026-09-03).** 토글 구현으로 이 조건이 뒤집혔다. 현재는 `@custom-variant dark (&:where([data-theme="dark"], [data-theme="dark"] *))`가 **있어야** 정상이다 — 없으면 토큰만 바뀌고 shadcn 컴포넌트의 `dark:` 유틸리티가 따라오지 않는다. 다만 shadcn 기본값인 `.dark` **클래스** 전략을 쓰지 않는다는 원래 취지는 유지된다(클래스가 아니라 `data-theme` 속성이다)
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
| 다크 전환 | Playwright `emulateMedia({colorScheme})`로 라이트↔다크 전환 후 computed style 측정 | 배경 `rgb(250,250,250)`↔`rgb(10,10,10)`, `--primary` `#4f46e5`↔`#818cf8`. `html`의 class는 두 상태에서 동일하고 `.dark`도 없다 — **CSS만으로 전환됨** <br>※ 2026-09-03 이후로는 `html[data-theme]`이 바뀌며 전환된다. 색값 자체는 그대로다 |
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

백엔드 `TodoResponse`의 `createdAt`·`updatedAt`·`completedAt`은 `LocalDateTime`이고 `application.properties`에 Jackson 날짜 설정이 없어 **`Z` 접미사 없이** `"2026-09-01T12:00:00.123456"` 형태로 직렬화된다. JS `new Date()`는 오프셋 없는 ISO 문자열을 **로컬 시각으로 해석**하므로 KST 브라우저에서 9시간 어긋난다. 백엔드를 고치지 않고 프론트 `lib/datetime.ts`의 `parseServerDateTime` 단일 진입점에서 `Z`를 붙여 흡수한다. `dueDate`는 `LocalDate`(`"2026-09-01"`)이므로 **`Z`를 붙이면 안 되며** 별도 함수로 분리한다.

> 버전·설치 방법은 `CLAUDE.md` 3장에서 모두 확정됐다. 이 Phase에서 재조사하지 않는다.

---

## Phase 7 — 인증 화면

> ### Phase 7 착수 결정 (2026-09-01)
>
> | 항목 | 결정 | 근거 |
> | --- | --- | --- |
> | 브랜치 | `develop`을 `main`으로 ff 정렬한 뒤 **`feature/` → `develop` → `main`** 복귀 | Phase 1~6이 `main`에 직접 커밋됐고 `develop`이 08-28에 멈춰 있었다. `develop` 전용 커밋이 0이라 무손실 정렬이 가능했다 |
> | 로그아웃 | **서버 `POST /api/auth/logout` 추가** | `CLAUDE.md` 5장(절대 규칙)·`PRD.md` F-08·7.4가 서버 로그아웃을 요구한다. `AuthService.logout()`이 이미 구현돼 있어 컨트롤러 메서드 1개 + `SecurityConfig` 1줄이면 된다 |
> | 네트워크 문구 | **"연결에 실패했습니다."로 통일** | `lib/errorMessages.ts`의 값을 이 문구로 바꾼다. 재시도 버튼이 옆에 붙으므로 "잠시 후 다시 시도해 주세요"는 중복이다 |
> | 구글 검증 | **백엔드를 8080에 띄워 실측** | 등록된 리다이렉트 URI가 `http://localhost:8080/login/oauth2/code/google`이고 `redirect-uri`를 따로 지정하지 않아 Spring이 baseUrl 기준으로 생성한다. 8081로 띄우면 `redirect_uri_mismatch`가 난다 |
>
> **참조 정정**: 이 Phase가 참조하던 `PRD.md` 5.1/5.3/5.4는 실제 문서와 달랐다(5.1=보안, 5.3=코드 품질, 5.4=접근성). 실제 내용은 7.1(가입 흐름)·7.2(구글 로그인)·7.3(토큰 갱신)·7.4(로그아웃)에 있어 그쪽으로 고쳤다. `CLAUDE.md` 9장 참조 2건은 해당 장이 없어 제거했다(`CLAUDE.md`는 6장까지). 추적표의 "JWT 24h"는 실제 30분(`access-token-expiration: 1800000`)이라 정정했다.
>
> ⚠️ **`UX-01`~`UX-07`은 `PRD.md`에 정의가 없다.** 위 추적표의 행 이름(UX-01 스켈레톤, UX-04 에러+재시도 버튼, UX-06 label·키보드)만이 유일한 근거다.

**저장소**: `todo-frontend` · **관련 요구사항**: AUTH-01~09, UX-01, UX-06 · **선행 조건**: 백엔드 **Phase 3 완료**(가입·로그인·`/auth/me`), 구글 로그인 DoD는 **Phase 5 완료** 필요. 백엔드를 로컬에서 띄운 상태로 진행한다

**작업**

- `/login`, `/signup`, `/oauth/callback`
- **`/oauth/callback`은 `useSearchParams`를 쓰므로 `<Suspense>`로 감싼다** (없으면 `npm run build` 실패)
- **`/oauth/callback` 세부** (`PRD.md` 7.2 구글 로그인 흐름)
  - 토큰 저장 → **URL에서 토큰 제거**(히스토리·공유 링크에 남지 않도록) → `/todos` 이동
  - **`token` 파라미터가 없거나 빈 문자열이면 `/login`으로 보낸다**
  - 처리 중에는 스켈레톤만 보여주고 **사용자가 조작할 요소를 두지 않는다**
  - **공통 헤더를 두지 않는다** (인증 처리 중 화면이라 닉네임을 알 수 없다)
- **`/todos` 플레이스홀더 페이지 생성** — 라우트 보호를 검증하려면 대상 페이지가 존재해야 한다 (내용은 Phase 8)
- Phase 6에서 비워둔 **헤더의 닉네임·로그아웃을 `useAuth`에 연결**
  - **이메일은 화면에 표시하지 않는다.** `/auth/me` 응답에는 들어오지만 헤더에는 닉네임만 노출한다 (`AUTH-08`)
- `useAuth` 훅 (로그인 / **로그아웃: `POST /api/auth/logout` + 토큰 삭제 + 캐시 초기화** / 현재 사용자)
- **라우트 보호는 `(main)` 클라이언트 레이아웃에서 처리한다. `middleware.ts`를 만들지 않는다** (localStorage는 middleware에서 읽을 수 없다)
  - **인증 판정이 끝나기 전에는 스켈레톤을 보여준다** (`UX-01`)
- 401 응답 시 자동 로그아웃 처리
- **`?error=email_conflict`와 `?error=oauth_failed` 안내 문구 표시** — 백엔드 `OAuth2FailureHandler`가 두 값을 보낸다. 하나만 처리하면 사용자가 취소했을 때 아무 안내도 없이 로그인 화면만 뜬다
- **`/signup` 실시간 검증** (`PRD.md` 7.1 — "검증 실패 시 해당 필드 아래 에러 표시") — 이메일 형식, **비밀번호 6자 이상 + UTF-8 72바이트 이하**, 닉네임 1~50자. 안내 문구에 **한글 1자 = 3바이트**임을 밝힌다
- **에러 문구는 Phase 6의 `lib/errorMessages.ts`만 사용한다** (정본은 백엔드 `ErrorCode` enum)
  - `INVALID_INPUT` → 서버가 준 필드별 메시지를 해당 입력 아래 인라인
  - `UNAUTHORIZED`(로그인 시) → "이메일 또는 비밀번호가 올바르지 않습니다."를 폼 상단 인라인
  - `UNAUTHORIZED`(그 외) → 문구 없이 `/login` 이동
  - `EMAIL_DUPLICATED` → "이미 사용 중인 이메일입니다."를 이메일 입력 아래 인라인
  - 네트워크 실패 → "연결에 실패했습니다." + 재시도 버튼

**DoD**

- [x] 이메일 가입·로그인 정상 동작
- [x] **가입 화면에서 한글 25자 비밀번호를 입력하면 제출 전에 바이트 초과 안내가 인라인으로 뜸** (서버 400에만 의존하지 않음 — `AUTH-02`)
- [x] **중복 이메일 가입 시 "이미 사용 중인 이메일입니다."가 이메일 입력 아래 인라인으로 뜸** (`AUTH-03`)
- [x] **미가입 이메일과 비밀번호 오류의 화면 문구가 동일함** ("이메일 또는 비밀번호가 올바르지 않습니다.") — 계정 존재 여부가 드러나지 않음
- [x] **백엔드를 내린 채 로그인을 시도하면 "연결에 실패했습니다." + 재시도 버튼이 나옴** (네트워크 실패 매핑 — `UX-04`)
- [x] 구글 로그인 → 콜백 → `/todos` 이동, URL에서 토큰 제거됨
- [x] **`/oauth/callback`에 `?token=` 없이 직접 접근하면 `/login`으로 이동함** (`PRD.md` 7.2)
- [x] **`/oauth/callback` 화면에 공통 헤더와 조작 가능한 요소가 없음**
- [x] **헤더에 닉네임이 보이고 이메일은 어디에도 렌더되지 않음** (DevTools에서 DOM 검색 — `AUTH-08`)
- [x] 계정 충돌 시 안내 문구 노출
- [x] 로그아웃 시 토큰·캐시 모두 제거되고 `/login`으로 이동
- [x] 새로고침해도 로그인 상태 유지
- [x] 미인증 상태로 `/todos` 접근 시 로그인으로 이동
- [x] **만료된 토큰을 localStorage에 직접 넣고 `/todos`에 접근했을 때, 보호 화면이 한 프레임도 노출되지 않고 곧바로 `/login`으로 이동**
  > `useAuth`가 토큰 존재 여부만 보면 만료 토큰이 판정을 통과해, 401 왕복 동안 보호 화면이 노출된다. `exp`를 디코드해야 한다. 검증용 만료 토큰은 `jwt.access-token-expiration`을 일시적으로 낮춰 발급받으면 된다
- [x] `middleware.ts` 파일이 존재하지 않음
- [x] `npm run build` 성공 (`useSearchParams` Suspense 경계 확인)
- [x] **모든 입력에 label 연결, Tab·Enter만으로 가입·로그인 완주 가능**

### Phase 7 재검증 / 발견 기록 (2026-09-01)

DoD 17개를 전부 실제 조작으로 확인했다. 최종 구성은 **백엔드 8080 · 프론트 3000**이다.

**검증 방법과 결과**

| 항목 | 확인 방법 | 결과 |
| --- | --- | --- |
| 빌드·정적검사 | `npm run check`, `npm run build` | 둘 다 종료코드 0 |
| 이메일 가입·로그인 | 브라우저에서 폼 제출 | 가입 → 즉시 로그인 → `/todos` |
| 한글 25자 비밀번호 | 입력 직후 DOM 조회 | `password-error`에 "현재 75바이트" 안내, **제출 버튼 `disabled`** |
| 영문 72/73자, 5자 | 동일 | 72자 통과 · 73자 차단 · 5자 차단 |
| 중복 이메일 | 기가입 이메일로 재가입 | `role=alert`가 **`email-error` 하나뿐**이고 문구는 "이미 사용 중인 이메일입니다." |
| 미가입/오답 문구 동일 | 두 경우 문구 문자열 비교 | 완전 일치 — "이메일 또는 비밀번호가 올바르지 않습니다." |
| 네트워크 실패 | 백엔드 종료 후 로그인 | "연결에 실패했습니다." + 재시도 버튼 |
| **구글 로그인** | 실제 구글 계정으로 완주 | `/todos` 도달, 헤더에 구글 계정 이름, **URL에 `token=` 흔적 없음**, 백엔드 에러 로그 0건 |
| 구글 `redirect_uri` | `curl -i /oauth2/authorization/google`의 Location 파싱 | `http://localhost:8080/login/oauth2/code/google` — 등록값과 일치 |
| 콜백 파라미터 방어 | `?token=` 없이 / 빈 문자열로 접근 | 둘 다 `/login` 이동 |
| 콜백 화면 구성 | 초기 HTML을 `fetch`로 받아 파싱 | header·button·a·input **전부 0개** (대조군 `/login`은 button 1·a 2) |
| 헤더 이메일 미노출 | `innerText`·`innerHTML` 문자열 검색 | 0건 |
| 로그아웃 | 버튼 클릭 후 Resource Timing 확인 | `/api/auth/logout` 요청 1건, 토큰 삭제, `/login` 이동 |
| 새로고침 유지 | `location.reload()` | `/todos` 유지, 헤더 닉네임 유지 |
| 미인증 `/todos` | 토큰 없이 접근 | `/login` 이동, 보호 화면 미렌더 |
| **만료 토큰** | `exp`가 과거인 JWT 주입 후 접근 | `/login` 이동, **백엔드 요청 0건**, 보호 화면 미렌더 |
| `middleware.ts` | `find` | 없음 |
| 키보드 완주 | Tab·Enter만으로 가입 | `/todos` 도달 |

> 만료 토큰 케이스가 핵심이다. Resource Timing에 **백엔드 요청이 0건**으로 찍혔다 — `useAuth`의 `enabled: isTokenValid(...)`가 `/auth/me` 호출 자체를 막아 401 왕복이 없다. 토큰 존재 여부만 봤다면 요청 1건이 찍히고 그 왕복 동안 화면이 노출됐을 것이다.

**발견해 고친 것**

1. **쿠키 없이 `POST /api/auth/refresh`가 500을 냈다.** `@CookieValue`가 `required=true`라 `MissingRequestCookieException`이 발생하는데 `GlobalExceptionHandler`가 그 타입을 처리하지 않아 generic 핸들러로 떨어졌다. 라우트 보호 동선에서 미인증 접근마다 발생하던 결함이다. 전용 핸들러를 추가해 **401**로 고쳤다.
2. **로그아웃은 서비스 계층이 이미 완성돼 있었다.** `AuthService.logout()`·`RefreshTokenService.revoke()`·`RefreshTokenCookieFactory.expire()`가 전부 존재했고 컨트롤러 엔드포인트만 빠져 있었다. 컨트롤러 메서드 1개와 `SecurityConfig` 1줄로 해결했다. 쿠키는 `required=false`로 받아 **멱등**하게 만들었다 — 쿠키가 없다는 것은 이미 로그아웃된 상태이지 오류가 아니다.
3. **`?error=oauth_failed` 처리가 누락돼 있었다.** ROADMAP 작업 목록이 `email_conflict`만 적었으나 `OAuth2FailureHandler`는 두 값을 보낸다. 하나만 처리하면 사용자가 구글 동의를 취소했을 때 아무 안내 없이 로그인 화면만 뜬다.

**미해결로 남긴 것**

- **잘못된 JSON 본문이 500을 낸다.** `HttpMessageNotReadableException`도 generic 핸들러로 떨어진다(검증 중 인코딩 문제로 우연히 확인했다). 400 `INVALID_INPUT`이 맞지만, 우리 프론트는 `JSON.stringify`로만 보내므로 Phase 7 동선에서는 발생하지 않아 손대지 않았다. 쿠키 결함과 같은 계열이다.
- **`docs/DESIGN.md`가 여전히 없다.** `shrimp-rules.md`는 이 문서를 "Phase 6 산출물"로 규정하지만 Phase 6은 팔레트를 ROADMAP 「Phase 6 확정 값」에 기록하는 것으로 갈음했다.

---

## Phase 8 — Todo 화면

> ### Phase 8 착수 결정 (2026-09-01)
>
> `PRD.md`와 이 문서가 서로 다른 말을 하던 5건을 확정했다. 결정하지 않았으면 만들 화면 자체가 달라졌을 항목들이다.
>
> | 항목 | 결정 | 근거 |
> | --- | --- | --- |
> | 정렬 UI | **생성일·마감일 2종 제공**, 우선순위 정렬은 없음 | `PRD.md` F-16이 정렬을 요구사항으로 명시한다. 이 문서가 "정렬 UI 없음"의 근거로 들던 **「`PRD.md` 1장 비목표」는 실재하지 않는다**(1장은 제품 원칙과 성공 기준뿐). 백엔드 `SORT_WHITELIST`가 `createdAt`·`dueDate` 둘뿐이라 2종은 서버 수정 없이 된다. 우선순위는 `Priority`가 enum이라 DB 정렬이 문자열 순(`HIGH<LOW<MEDIUM`)이 되어 의미가 맞지 않는다 |
> | 페이지 크기 | **백엔드에 상한 50을 넣고 기본값 10을 명시**, 프론트도 `size=10` 전송 | 실측 결과 **기본값은 이미 10이었다** — `@PageableDefault`의 `size()` 기본값이 10이다(`spring.data.web.pageable.default-page-size`의 20과 혼동하기 쉽다). 문제는 **상한**이다. `size=10000`을 보내면 Spring 기본 `max-page-size` 2000으로만 잘려 `PRD.md` F-13(최대 50)·NF-08(서버가 제한)을 위반한다 |
> | 수정 메서드 | **PUT 유지**, `PRD.md` F-19를 정정 | 백엔드·Phase 4 DoD·이 Phase가 전부 PUT이고 Phase 4에서 이미 검증을 통과했다. 폼이 네 필드를 항상 함께 보내므로 부분 수정의 실익이 없다. PATCH는 nullable 필드와 "값을 null로 지우기"를 구분하기 어려워 마감일 삭제 처리가 애매해진다 |
> | 삭제 확인 | **만든다** (목록·상세 양쪽) | `PRD.md` F-22가 명시 요구인데 이 Phase의 작업 목록·DoD 어디에도 없었다. `alert-dialog`를 어차피 받으므로 비용이 낮다. Soft Delete라 데이터는 남지만 **UI상 복구 경로가 없어 사용자에겐 사실상 비가역**이다 |
> | 요약 카운트 뱃지 | **Phase 8에서는 미룬다** | `PRD.md` F-23이 요구하지만 백엔드에 카운트 엔드포인트가 없다(컨트롤러·DTO grep 확인). 신규 API(컨트롤러·서비스·리포지토리·DTO)가 필요해 이 Phase 범위가 크게 는다. 필터 버튼은 숫자 없이 만들고 **F-23은 Phase 9 이후 항목으로 이관**한다 |
>
> **참조 정정**: 이 Phase가 참조하던 `PRD.md` 5.5·5.6은 실재하지 않는다(5장은 5.1 보안 / 5.2 데이터 / 5.3 코드 품질 / 5.4 접근성까지). `CLAUDE.md` 6·8·9장 참조 3건도 해당 장이 없어 제거했다(`CLAUDE.md`는 6장까지이고 6장은 「작업 방식」이다). `UX-01`~`UX-06`은 `PRD.md`에 정의가 없고 위 추적표의 행 이름만이 근거다.
>
> **시드 상태(2026-09-01 확인)**: `seed-dev@example.com`이 Todo **100건**(완료 33건, 마감일 없음 25건)으로 이미 적용돼 있다. `seed-perf@example.com`은 10,000건이다. **`seed-dev.sql`의 `todos` INSERT는 멱등이 아니므로** 재실행 전에 반드시 건수를 먼저 센다.
>
> **부수 발견(Phase 2 소관, 이 Phase에서 고치지 않음)**: `User.email`이 `@Column(unique = true)` **전체 유니크**라 `PRD.md` NF-13(「살아있는 사용자 기준 부분 유니크 인덱스」)과 F-02(「Soft Delete된 계정의 이메일은 재사용 가능」)를 만족하지 못한다. 지금은 삭제된 계정의 이메일을 영원히 재사용할 수 없다.

**저장소**: `todo-frontend` · **관련 요구사항**: TODO-01~10, 12~16, UX-01~06 · **선행 조건**: 백엔드 **Phase 4 완료**(Todo API 6종 + 시드). `db/seed-dev.sql`을 로컬에 적용한 상태로 진행해야 페이지네이션·필터 DoD를 눈으로 확인할 수 있다

**작업**

- `/todos`: 목록, 검색, 완료 필터, 페이지네이션
- **검색어·필터·페이지는 URL 쿼리로 관리** (`?page=2&completed=false&keyword=...`)하고, 페이지 전체를 `<Suspense>`로 감싼다
- `useTodos` 훅 (React Query)
- **`TodoForm` 공용 컴포넌트** → `/todos/new`와 `/todos/[id]`가 재사용 (진입 즉시 편집 가능, 명시적 저장)
- **`TodoForm`에 완료 체크박스를 두지 않는다.** 완료는 목록에서만 변경
- **Tiptap 통합 — StarterKit을 기본값으로 쓰지 않는다.** `heading.levels [2,3]`, `strike: false`, `horizontalRule: false`, **`underline: false`**로 설정하고 **`link`는 StarterKit 내장 옵션으로 설정**한다 (v3에서 Link·Underline이 StarterKit에 포함됨 — 공식 문서 확인)
  > ⚠️ `setContent`는 기본으로 `onUpdate`를 발생시킨다(`emitUpdate` 기본 `true`). 초기 주입에서 그냥 호출하면 폼이 즉시 dirty로 표시된다. **정규화·TrailingNode와 별개인 두 번째 원인**이므로 `{ emitUpdate: false }`를 함께 줘야 한다.
  > ⚠️ `useEditor`에 **`immediatelyRender: false`**가 필수다. Next.js는 기본이 SSR이라 이 옵션 없이는 하이드레이션 불일치 에러가 난다.
- **`editor.commands.setContent()` 호출 직전에 `lib/sanitize.ts`로 DOMPurify 정화** — 이 앱에는 `dangerouslySetInnerHTML`이 없으므로 여기가 유일한 렌더 방어 지점이다
- 우선순위 뱃지, 마감일 표시 (date-fns 포맷)
- **완료 항목은 제목에 취소선 + 흐린 색상**을 적용한다. 색만으로 구분하지 않는다(체크박스 상태와 취소선이 함께 신호를 준다 — `PRD.md` NF-24)
- **`/todos/[id]` 저장 실패 처리** — 폼 내용을 유지한 채 에러를 표시한다. 입력을 날리거나 목록으로 튕기지 않는다 (`TODO-13`, `UX-04`)
  > ⚠️ `TODO-13`은 Phase 9의 토글·삭제 롤백만으로 충족되지 않는다. **저장(PUT) 실패 경로는 낙관적 업데이트를 쓰지 않으므로 Phase 9가 손대지 않는다.** 이 Phase에서 별도로 처리한다.
- **`/todos/[id]` 삭제 성공 시 `/todos`로 이동**한다 (`TODO-12`)
- **삭제 전 확인 다이얼로그를 거친다** (`PRD.md` F-22). 목록·상세 양쪽에서 적용한다
- **에러 문구는 Phase 6의 `lib/errorMessages.ts`만 사용한다.** `NOT_FOUND`는 전체 화면 상태, `INTERNAL_ERROR`는 토스트 또는 에러 카드, 네트워크 실패는 재시도 버튼이 있는 에러 카드 (백엔드 `ErrorCode` enum)
- **이탈 확인 대화상자 — 3계층으로 구현** (`beforeunload` + 버튼 핸들러 + `popstate` 가드). App Router에 공식 차단 API가 없어 한 줄로 끝나지 않는다. **별도 공수 4~8시간을 잡는다**
- **`dirty` 판정을 직접 구현한다.** 폼 라이브러리를 쓰지 않으므로 `formState.isDirty`가 없다. 제목·우선순위·마감일은 단순 비교로 끝나지만 **본문은 그렇지 않다** — Tiptap이 HTML을 자기 스키마로 정규화하므로 서버 원본과 `editor.getHTML()`을 직접 비교하면 사용자가 아무것도 고치지 않아도 dirty로 판정된다. **초기 스냅샷은 `setContent()` 직후의 `editor.getHTML()`로 잡는다**(정규화를 거친 값끼리 비교)
- **삭제 실패 시 페이지 이동까지 되돌린다** — 페이지 이동은 `onMutate`가 아니라 `onSuccess`에서 수행
- **경계 상황**: 마지막 항목 삭제로 페이지가 비면 이전 페이지로 이동, `/todos/[id]` 404 시 전용 화면
- 로딩 스켈레톤 / 빈 상태 / 검색 결과 없음 / 에러 상태

**DoD**

- [x] **목록이 페이지당 정확히 10건씩 끊기고, 항목 순서가 생성일 내림차순임** (시드 데이터의 `created_at`과 대조 — `TODO-05`, `TODO-06`)
- [x] 20건 이상에서 페이지네이션 정상
- [x] **서버가 페이지 크기를 제한함** — `size` 미지정 시 10건, `size=51`이 50으로 클램프됨 (`PRD.md` F-13·NF-08)
- [x] **정렬 2종(생성일·마감일)이 동작하고 오름·내림차순을 고를 수 있음. 우선순위 정렬은 제공하지 않음** (백엔드 화이트리스트가 `createdAt`·`dueDate` 둘뿐이다)
- [x] **정렬 상태가 URL에 반영되고 새로고침·뒤로가기에서 유지됨** (`PRD.md` F-17)
- [x] 검색·필터 상태가 URL에 반영되고 새로고침·뒤로가기에서 유지됨
- [x] **완료 처리한 항목의 제목에 취소선과 흐린 색상이 적용됨**
- [x] `npm run build` 성공
- [x] Tiptap 내용 저장 후 재조회 시 서식 유지 (툴바 항목 전부)
- [x] 본문에 `# `, `~~취소선~~`, `---`를 입력해도 서식이 생성되지 않음 (저장 후 소실되는 입력이 없음)
- [x] **본문에서 `Ctrl+U`를 눌러도 밑줄(`<u>`)이 생성되지 않음** (v3 StarterKit의 Underline이 꺼져 있는지 확인 — 켜져 있으면 저장 시 서식이 조용히 사라진다)
- [x] 2페이지 이상에서 마지막 항목 삭제 시 이전 페이지로 이동
- [x] **2페이지 마지막 항목 삭제가 실패했을 때, 사용자가 보고 있는 화면에서 롤백이 눈으로 확인됨** (페이지가 먼저 넘어가 버려 롤백이 안 보이면 실패)
- [x] 타인 소유 id로 접근 시 "찾을 수 없습니다" 화면 표시 + **"목록으로 가기" 버튼이 있고, 자동 리다이렉트가 일어나지 않음** (`TODO-14`, `PRD.md` 5.6)
- [x] `/todos/[id]` 로딩 중 **폼 형태 스켈레톤**이 보임 (`UX-01`)
- [x] **백엔드를 내린 채 저장을 누르면 입력한 제목·본문이 그대로 남아 있고 에러가 표시됨** (`TODO-13`, `UX-04`)
- [x] **`/todos/[id]`에서 삭제하면 `/todos`로 이동함** (`TODO-12`)
- [x] **`setContent()` 직전에 `lib/sanitize.ts`가 호출됨** (본문에 `<script>`가 섞인 데이터를 DB에 직접 넣고 상세 화면 진입 시 실행되지 않음)
- [x] `/todos/new`와 `/todos/[id]`가 `TodoForm`을 재사용
- [x] 수정 화면에서 저장해도 완료 상태가 바뀌지 않음
- [x] 변경 후 이탈 시 확인 대화상자 노출 (**새로고침 / 페이지 내 취소 버튼 / 브라우저 뒤로가기 3경로 모두**)
- [x] **저장 직후에는 확인 대화상자가 뜨지 않음** (`dirty` 해제 확인)
- [x] **본문이 있는 할 일을 열어 아무것도 고치지 않고 나갈 때 확인 대화상자가 뜨지 않음** (Tiptap 정규화로 dirty가 오판되지 않는지 — 서식이 섞인 본문으로 시험한다)
- [x] 4가지 화면 상태 모두 눈으로 확인
- [x] **에러 상태의 재시도 버튼을 눌렀을 때 실제로 재요청이 나가고, 서버를 다시 올리면 목록이 정상 렌더됨** (`UX-04` — 버튼이 보이기만 하고 동작하지 않는 경우를 걸러낸다)
- [x] **빈 상태 문구("아직 할 일이 없어요")와 검색 결과 없음 문구가 서로 다름** (`UX-03`)
- [x] 360px 화면에서 레이아웃 정상 (가로 스크롤 없음)
- [x] **키보드만으로 할 일 생성·완료 토글·삭제 수행 가능**

> ### Phase 8 재검증 / 발견 기록 (2026-09-01)
>
> DoD 28항목을 실제로 돌려 확인했다. 그 과정에서 **결함 2건을 찾아 고쳤고, 내가 내린 오진 2건을 되돌렸다.**
>
> | # | 발견 | 조치 |
> | --- | --- | --- |
> | 1 | **페이지네이션이 360px에서 넘쳤다.** `PRD.md` F-29의 「모바일에서는 더 축약」이 구현돼 있지 않았다. 버튼 7개 × 44px + 간격이 가용 폭을 넘었다 | 현재 페이지에서 2칸 이상 떨어진 번호에 `hidden sm:inline-flex`를 적용. 360px 실측에서 페이지네이션 **309px / 가용 341px**, 가로 스크롤 없음 |
> | 2 | **오프라인·백그라운드에서 목록이 조용히 어긋났다.** React Query는 요청을 시작·재개할 수 없으면 `fetchStatus`를 `paused`로 두는데, 이때 `status`는 그대로라 `isError`가 켜지지 않는다. 결과적으로 **URL은 4페이지인데 화면에는 1페이지 데이터가 남고 아무 경고도 없었고**, 첫 진입이면 **스켈레톤이 영원히 돌았다** | `query.isPaused`로 분기를 추가했다. 데이터가 있으면 목록을 유지한 채 경고 배너 + 재시도 버튼, 없으면 에러 화면을 띄운다. 온라인 복귀 시 일시정지된 요청이 자동 재개되는 것까지 실측 확인 |
> | 3 | ~~데이터가 있는 상태에서 재요청이 실패하면 `isError`가 켜지지 않으므로 `failureCount`로 감지해야 한다~~ | **오진. 되돌렸다.** `query-core/query.js`의 리듀서는 `case "error"`에서 **데이터 유무와 무관하게 `status: "error"`를 무조건 설정**한다. 따라서 `!isError && failureCount > 0` 조건은 실패 시 참이 될 수 없는 죽은 코드였다. 실제로 막아야 했던 것은 위 2번의 `paused`였고, 그 경로에서는 시도 자체가 없어 `failureCount`가 **0**이라 원래 조건으로는 배너가 뜨지 않았다 |
> | 4 | ~~재시도 버튼이 요청을 내지 않으니 `networkMode: "always"`가 필요하다~~ | **오진. 되돌렸다.** `retryer.js`의 `canContinue()`는 `focusManager.isFocused()`를 함께 보고, `focusManager`는 `document.visibilityState !== "hidden"`으로 판정한다. 원격 제어 중인 백그라운드 Chrome 창이 `hidden`이라 재시도가 **의도대로** 멈춘 것이었다. 창을 앞으로 꺼내 `visible` 상태에서 재측정하니 **클릭 1회당 요청 정확히 1건**이 나갔고 3페이지가 정상 렌더됐다. `networkMode`는 `refetchOnReconnect` 동작까지 바꾸므로 근거 없이 남길 수 없어 2개 파일에서 제거했다 |
>
> **교훈**: 3·4번 모두 **코드를 읽고 세운 가설을 실측으로 확인하지 않은 채 고친 것**이 원인이다. 라이브러리 동작은 추측하지 말고 `node_modules`의 실제 소스를 열어 확인한다.
>
> **검증 방법**: 360px는 Chrome 최소 창 너비(약 500px) 제약 때문에 창 크기로는 잴 수 없어, **동일 출처 360px `<iframe>`** 안에서 측정했다(iframe 내부 문서는 자체 뷰포트를 가지므로 미디어 쿼리가 실제 기기와 동일하게 평가된다). 목록·작성·상세 세 화면 모두 넘치는 요소 0개, 44px 미만 터치 타깃 0개. 네트워크 실패는 `window.fetch` 가로채기로, 오프라인은 `offline` 이벤트 디스패치로 재현했다(둘 다 앱 코드를 건드리지 않는다).
>
> **검증 데이터 정리**: XSS 검증용으로 DB에 직접 넣은 Todo와 페이지네이션 검증용 계정(`p8page@example.com`) 및 그 항목 10건을 **Soft Delete로** 정리했다(규칙 5에 따라 물리 삭제하지 않았다). 사용자 소유 항목은 `deleted_at IS NULL` 조건으로 제외해 손대지 않았다.

---

## Phase 9 — 인터랙션 다듬기

**저장소**: `todo-frontend` · **관련 요구사항**: TODO-11~13 · **선행 조건**: Phase 8 완료(목록·상세 화면이 실제 API로 동작하는 상태). 백엔드 Phase 4의 `toggle` 멱등 동작이 전제다

> `TODO-13` 중 **저장(PUT) 실패 처리는 Phase 8에서 끝낸다.** 이 Phase가 다루는 것은 낙관적 업데이트를 쓰는 **토글·삭제**의 롤백뿐이다.

> ### Phase 9 착수 결정 (2026-09-03)
>
> **연타 직렬화 방식: `scope`로 확정한다 (`isMutating()` 가드 아님).** 이전 버전이 근거로 들던
> **「`CLAUDE.md` 9장」은 실재하지 않는다**(`CLAUDE.md`는 6장 「작업 방식」까지다 — 과거 Phase들에서
> 반복 발견된 깨진 크로스레퍼런스와 같은 종류의 오기). 실제 근거는 TanStack Query 소스다:
> `node_modules/@tanstack/query-core`의 `retryer.js`를 보면 `scope`가 거는 `canRun()`은
> `config.fn()`(=실제 mutationFn 호출, fetch 발사) **이전에** 확인되어 조건이 거짓이면 `pause()`로
> 들어간다 — 즉 `scope`는 **네트워크 요청이 서버에 도달하는 순서 자체**를 직렬화한다. 반면
> `onSettled`의 `isMutating()` 가드는 응답이 이미 온 뒤에야 "이게 마지막 요청인지" 판정하므로
> **네트워크 재정렬**(요청이 보낸 순서와 다르게 서버에 도착하는 것)까지는 막지 못한다. `scope`는
> `useMutation` 정의 시점의 정적 옵션이라 페이지 레벨 단일 훅 공유로는 todo별로 줄 수 없어,
> `useToggleTodo(id)`로 시그니처를 바꿔 `TodoItem` 내부에서 직접 호출하도록 리팩터링했다(호출부는
> `TodoItem` 1곳뿐 — Phase 8에서 `TodoForm`에 완료 체크박스를 두지 않기로 했으므로 영향 범위가 좁다).
>
> **삭제는 scope를 쓰지 않는다.** 확인 다이얼로그가 이미 연타를 막으므로(확인 버튼이 대기 중
> 비활성화) 페이지 레벨 단일 인스턴스 구조를 유지했다.

**작업**

- 완료 토글·삭제 낙관적 업데이트 (`onMutate` / `onError` 롤백 / `onSettled`)
- 토글은 **목표 상태를 그대로 서버에 전송** (서버 계산에 의존하지 않음)
- **연타 대비 — mutation 직렬화.** React Query v5 mutation은 기본 병렬이라 목표 상태 전송(멱등)만으로는 요청 재정렬을 막지 못한다. `scope: { id: \`todo-toggle-${todoId}\` }`를 적용한다(위 착수 결정 참고)
- Motion: 목록 등장(stagger), 삭제(`AnimatePresence`), 토글 스프링 — **import는 `motion/react`**
- 실패 시 토스트 알림(`sonner`)
- `prefers-reduced-motion` 대응

**DoD**

- [x] 토글·삭제 시 대기 시간 없이 즉시 반영
- [x] **체크박스를 빠르게 연타해도 새로고침 후 상태가 UI와 일치**
- [x] **연타를 멈춘 뒤 최종 상태가 "마지막에 클릭한 값"과 일치하며, 잠시 후 반대 값으로 되돌아가지 않음**
  > 앞 항목만 보면 결함이 통과한다. `invalidateQueries`가 어떤 값으로든 수렴시키므로 "UI와 서버가 일치"는 항상 참이 된다. 문제는 **수렴한 값이 사용자 의도와 다를 수 있다는 것**이다
- [x] 서버를 내린 상태에서 실패 → UI 롤백 + 알림 확인
- [x] 애니메이션이 200ms 이내, 과하지 않음
- [x] 애니메이션 관련 import가 모두 `motion/react`에서 이루어짐

> ### Phase 9 재검증 / 발견 기록 (2026-09-03)
>
> DoD 6항목을 브라우저에서 정밀 폴링(50ms 간격)으로 실측했다.
>
> - **연타 직렬화 증명**: 체크박스 5회 연타 후 Resource Timing API로 5건의 `PATCH /toggle` 요청의
>   `startTime`/`responseEnd`를 측정해 **겹침 0건**을 확인했다 — `scope`가 요청 발사 순서를 실제로
>   직렬화한다는 것을 코드가 아니라 관측값으로 증명했다. 새로고침 후 서버 상태도 마지막 클릭값과
>   일치했다.
> - **롤백 + 토스트 타이밍**: 서버를 내린 상태에서 토글은 t=477ms에 낙관적 반영 → t=2427ms에
>   실패 확정과 동시에 롤백 + 토스트, 삭제는 t=305ms 낙관적 반영 → t=2494ms 롤백 + 토스트. 로컬호스트에서
>   connection-refused가 확정되기까지 약 1.8~2초 걸리는 것은 이 개발 환경(Windows)의 특성이며
>   앱 결함이 아니다(직접 `fetch`로 실측: 1799ms). 이 지연 동안 낙관적 업데이트가 화면에 계속
>   유지되는 것은 의도된 동작이다.
> - **reduced-motion**: 이 세션에서 OS 레벨 `prefers-reduced-motion` 에뮬레이션 도구를 쓸 수 없어
>   실제 토글 관측 대신 `motion-dom` 소스(`visual-element-target.mjs`)로 대체 검증했다. `reducedMotion`
>   활성화 시 `positionalKeys`(transform 계열 — 이 Phase가 쓰는 `y`, `scale` 포함)만 즉시 처리되고
>   `opacity`는 그대로 전환되는데, 이는 전정기관을 자극하는 이동·확대만 끄고 안전한 페이드는
>   남기는 WCAG 권고와 정확히 일치하는 설계다.
> - **문서 오류 정정**: 이 절이 근거로 들던 `CLAUDE.md` 9장 인용(실재하지 않는 장)을 위 착수
>   결정으로 대체했다.

---

## Phase 10 — 전체 검증

**저장소**: 전체 · **새 테스트를 작성하지 않는다.** 전체 통과와 아래 체크리스트만 확인한다.

**작업**

- 각 저장소 README 작성
- 전체 검증 체크리스트 수행

### 최종 검증 체크리스트 (완료 판정 정본)

**환경**

- [x] `todolist_db`, `todolist_db_test`와 함께 PostgreSQL 실행
- [x] `./mvnw spring-boot:run` 오류 없이 기동
- [x] `./mvnw test` 전체 통과 (통합 테스트 8건 + Repository 단위 테스트) — 실측 결과 19건: 통합 8(Auth 3·Todo 4·스모크 1) + Repository 8(Todo 4·User 4) + 서비스 3(OAuth2). 원래 "8건" 표현은 정확했다
- [x] `npm run build` 성공
- [x] Swagger UI에서 전체 API 확인 — 실측 중 `/swagger-ui.html`이 401로 막혀 있는 결함을 발견해 즉시 수정(아래 발견 기록 참고)
- [x] 세 저장소의 브랜치가 `main`/`develop` 체계이고 `master`가 남아 있지 않음
- [x] `CLAUDE.md`·`PRD.md`·`ROADMAP.md`가 서로를 참조하는 경로가 실제 파일 위치와 일치함 — 이 과정에서 이 문서 자체의 오기 5건(PRD 5.1 에러 문구 매핑 오기 2건, CLAUDE.md 6장 오인용 3건)을 추가로 발견해 정정했다

**인증**

- [x] 회원가입 시 사용자 생성 및 JWT 반환
- [x] 로그인 시 유효한 JWT 반환 (`sub`에 user id)
- [x] 보호된 엔드포인트에 유효 토큰 필요
- [x] 구글 소셜 로그인 정상 동작, nickname 채워짐 — 실제 구글 계정 로그인은 자격증명 대리 입력 정책상 자동화 불가. `/oauth2/authorization/google` 리다이렉트 배선과 `CustomOAuth2UserServiceTest`(신규 계정 생성 시 nickname이 구글 name으로 채워짐을 검증하는 단위 테스트, Task 2에서 통과 확인)로 대체 확인
- [x] 동일 이메일 로컬 계정 존재 시 구글 로그인 거부 및 안내 — 위와 동일한 이유로 `CustomOAuth2UserServiceTest`의 이메일 충돌 거부 테스트(OAuth2AuthenticationException + EMAIL_CONFLICT_ERROR_CODE)로 대체 확인
- [x] 로그아웃 시 토큰·캐시 제거
- [x] 헤더에 닉네임만 표시되고 이메일은 화면 어디에도 노출되지 않음 (`AUTH-08`)
- [x] 만료 토큰으로 보호 화면 접근 시 화면 노출 없이 `/login`으로 이동 (`AUTH-07`)

**기능**

- [x] Todo CRUD가 페이지네이션과 함께 작동
- [x] 모든 응답이 `{success, data, error}` 포맷 (목록 포함)
- [x] 완료 필터(미지정 시 전체)·제목 검색(대소문자 무시) 동작
- [x] 수정 저장이 완료 상태를 덮어쓰지 않음 — 토글로 완료 처리 후 PUT으로 나머지 필드를 전부 바꿔도 completed·completedAt이 그대로 유지됨을 확인
- [x] 토글 연타 후에도 서버 상태와 UI 일치
- [x] Soft Delete 시 `deleted_at` 갱신 및 목록 제외 — DB에 행은 남고 목록에서만 제외됨(물리 삭제 아님)까지 확인
- [x] 타 사용자 리소스 접근 시 404 — GET·PUT·DELETE 세 메서드 전부 404(403 아님) 확인
- [x] Tiptap 저장/렌더링 정상, 우선순위·마감일 반영
- [x] 낙관적 업데이트 및 실패 롤백 동작

**보안**

- [x] `<script>` 포함 본문이 저장 시 정화됨 — Jsoup을 우회해 DB에 직접 삽입 후 렌더 화면에서도 DOMPurify가 실행을 막음을 확인(이중 방어 양쪽 다 실측)
- [x] 링크에 `rel="noopener noreferrer"` 주입됨 — **저장 시(Jsoup)뿐 아니라 렌더 후 DOM에서도 남아 있는지 확인.** DOMPurify의 `ALLOWED_ATTR`에 `rel`·`target`이 없으면 렌더 단계에서 지워진다
- [x] **`setContent()` 직전에 DOMPurify가 적용됨** (이 앱에는 `dangerouslySetInnerHTML`이 없으므로 여기가 유일한 렌더 방어 지점 — `CLAUDE.md` 3장 절대 규칙 8)
- [x] **툴바 · Tiptap 확장 · Jsoup 화이트리스트 · DOMPurify `ALLOWED_TAGS` 네 곳의 태그 집합이 일치** — `@tiptap/starter-kit`의 실제 번들 확장 목록을 `.d.ts`로 직접 확인해 4곳 모두 `p h2 h3 strong em ul ol li blockquote pre code br a` 13종으로 완전히 일치함을 확정(아래 비교표 참고)
- [x] 입력값 상한(비밀번호 **UTF-8 72바이트**, 제목 200자, 본문 50,000자) 검증 동작 — **한글 비밀번호로도 시험한다** — 한글 25자(75바이트)로 72바이트 초과 거부, 제목 201자·본문 50001자도 각각 정확히 거부됨을 확인
- [x] 시크릿이 저장소에 커밋되지 않음 — 세 저장소 `git log --all` 전체 이력에서 시크릿 파일·값 커밋 이력 0건

**UX**

- [x] 로딩/빈 상태/검색 결과 없음/에러 상태 모두 확인 (**에러 상태의 재시도 버튼이 실제로 재요청을 보냄** — `UX-04`)
- [x] `lib/errorMessages.ts`의 에러 문구 매핑이 화면에서 표 그대로 나옴 — **7종 전부 화면에서 확인 (2026-09-03 후속).** 단, 확인 경로가 두 종류이며 아래 「에러 문구 매핑 7종」 표에 어느 쪽인지 명시했다. 5종은 실제 서버 응답으로, `FORBIDDEN`·`RESET_TOKEN_INVALID` 2종은 서버가 그 코드를 던지는 경로 자체가 없어 응답을 주입해 **프론트 매핑 경로만** 확인했다
- [x] 로그인 실패 문구가 계정 존재 여부를 구분하지 않음
- [x] 360px ~ 1920px 반응형 정상 — 360px는 Phase 8, 1920px는 이번에 확인(넘침 없음, 콘텐츠 폭 적절히 제한)<br>⚠️ **이 판정은 짧은 닉네임에서만 성립하고 있었다(2026-09-03 후속 발견).** 닉네임은 50자까지 허용되는데 헤더에 축소 처리가 없어 긴 닉네임에서 320px를 넘쳤다. `min-w-0` + `truncate`로 수정 후 재실측: 닉네임 50자·컨테이너 320px에서 닉네임 605px→63px 축소·말줄임 적용, 로고 64px·테마 54px·로그아웃 90×44px 유지, 가로 넘침 없음
- [ ] **Chromium 계열 1종 + 사용 가능한 다른 엔진 1종에서 확인** ~~(Mac은 Chrome+Safari, Windows는 Chrome+Edge/Firefox)~~ — **이 안내 자체가 부정확했다(2026-09-03 정정).** **Edge는 Chromium(Blink) 엔진이라 "다른 엔진"이 아니다.** Chrome+Edge를 확인해도 같은 엔진을 두 번 보는 것이므로 이 항목의 목적(렌더링 엔진 차이 검증)을 달성하지 못한다. Windows에서 실질적인 두 번째 엔진은 **Gecko(Firefox)** 뿐이다.<br>이 PC 실측: Firefox 미설치, Playwright MCP도 Blink(`Chrome/152.0.0.0`, `vendor: Google Inc.`), Playwright 브라우저 캐시에 Firefox·WebKit 바이너리 없음. **다른 엔진이 존재하지 않아 이 환경에서는 충족 불가**다. Firefox 설치는 사용자 결정 사항이라 임의로 하지 않는다.<br>**결정 (2026-09-07): Phase 11 배포 후 실기기로 확인.** 지금 Firefox를 설치하지 않고, Amplify/EC2에 실제로 배포된 뒤 다른 브라우저(모바일 Safari, 실PC의 Firefox 등)로 접속해 확인하기로 했다 — Phase 11 DoD에 이관
- [x] OS 다크 설정에 따라 테마 전환 — Windows 레지스트리를 임시 전환해 배경색이 실제로 바뀌는 것을 실측 후 원복 (2026-09-03 토글 도입 후 재실측: "시스템" 상태에서 OS 다크 + 새로고침 시 `data-theme="dark"`, 배경 `rgb(10,10,10)` 확인)
- [x] 폼 label 연결 및 키보드 조작 가능

→ 전 항목 통과 시 세 저장소에 `v1.0.0` 태그. **37개 중 36개 통과 (2026-09-03 후속 검증 반영).** 남은 1개는 두 번째 렌더링 엔진 항목으로, **이 PC에 Chromium 외의 엔진이 설치되어 있지 않아 환경상 충족 불가**다(Firefox 설치가 선행되어야 한다). **결정 (2026-09-07): Firefox를 지금 설치하지 않고 Phase 11 배포 후 실기기로 확인하기로 했다 — Phase 11 DoD에 이관.** 이 결정으로 Phase 10은 ✅로 표시하되, `v1.0.0` 태그는 아직 걸지 않았다(이연된 항목이 남아 있어 태깅 시점은 별도로 확인이 필요하다).

> ### Phase 10 재검증 / 발견 기록 (2026-09-03)
>
> 계획 단계에서 "39개"로 셌던 체크리스트는 실제로 **37개**다(환경 7 · 인증 8 · 기능 9 · 보안 6 · UX 7). ROADMAP 원문 자체는 정확했고 계획 당시 집계가 틀렸다.
>
> **4곳 태그 집합 비교표** (보안 그룹에서 가장 결함 가능성이 높다고 판단한 항목 — 결과: 완전 일치)
>
> | 소스 | 태그 |
> |---|---|
> | Jsoup Safelist | `p h2 h3 strong em ul ol li blockquote pre code br a` |
> | DOMPurify `ALLOWED_TAGS` | `p h2 h3 strong em ul ol li blockquote pre code br a` |
> | Tiptap 실제 활성 확장 (`@tiptap/starter-kit`의 `.d.ts`로 직접 확인, strike·underline·horizontalRule 비활성화 반영) | 동일 13종 |
> | 툴바 버튼 | 위 13종 중 `p`(기본값)·`li`(목록의 암시적 자식)·`br`(Shift+Enter)를 제외한 나머지에 버튼 존재 — 3개는 버튼이 없는 게 정상(암시적/키보드 접근) |
>
> **백엔드 테스트 실측 개수**: `./mvnw test` 19건 전부 통과. Surefire 리포트에 구 패키지(`com.example.*`, 8/28 잔존) 3건이 섞여 있어 타임스탬프로 걸러냈다. 현재 패키지 6개 클래스: `AuthControllerTest` 3 · `TodoControllerTest` 4 · `TodoRepositoryTest` 4 · `UserRepositoryTest` 4 · `CustomOAuth2UserServiceTest` 3 · `TodoBackendApplicationTests`(스모크) 1. 원래 "통합 테스트 8건" 표현(Auth 3+Todo 4+스모크 1)은 정확했다.
>
> **실측 중 발견해 즉시 수정한 결함 3건**
>
> 1. **Swagger UI 진입점이 401로 막혀 있었다.** `SecurityConfig`의 `PERMIT_ALL_PATHS`에 `/swagger-ui/**`만 있고 SpringDoc의 실제 진입점 `/swagger-ui.html`(→ `/swagger-ui/index.html` 리다이렉트)이 빠져 있었다. 경로 하나를 추가해 재기동 후 정상 렌더(2개 태그, 11개 오퍼레이션) 확인.
> 2. **`CustomOAuth2UserServiceTest`의 주석이 실재하지 않는 「`CLAUDE.md` 14장」을 인용**하고 있었다(`CLAUDE.md`는 6장까지). 코드베이스 전체(백엔드·프론트엔드)를 grep해 이 1건 외에는 3~5장(유효 범위)만 있음을 확인 후 제거.
> 3. **`TodoForm.tsx`가 "Cannot update a component while rendering a different component" 경고를 내고 있었다**(사용자가 실사용 중 발견해 보고). 렌더 중 상태 조정 패턴을 자기 자신이 아니라 부모(`NewTodoPage`/`[id]/page.tsx`)의 `setIsDirty`에 적용한 것이 원인 — `useEffect`로 옮겨 수정. `reportedDirty` 보조 state도 함께 제거됐다.
>
> **문서 오류 추가 발견 5건**: `docs/ROADMAP.md` 110·111번째 줄이 "PRD.md 5.1의 에러 문구 매핑 표"(존재하지 않음)와 "PRD.md 7장 비기능요구사항"(실제로는 5장)이라는, 이 문서 전체에서 가장 이른 지점의 오기였다. 253·302·827번째 줄의 "CLAUDE.md 6장" 3건도 실제 근거인 3장 절대 규칙 8로 정정했다. 이미 완료·병합된 Phase 3·7·8 절까지 포함해 함께 고쳤다(Phase 9에서도 같은 패턴을 발견해 과거 Phase 절까지 정정한 전례를 따름).
>
> **미해결 — 범위가 큰 결함 (사용자 보고 완료, 기록만 남기고 넘어가기로 결정, 2026-09-03)**
>
> `RESET_TOKEN_INVALID` 에러 코드가 enum에만 정의되어 있고 실제로 던지는 코드가 없다. 조사 결과 **비밀번호 재설정 기능이 백엔드·프론트엔드 양쪽 다 미구현**이다 — `PasswordResetToken` 엔티티·Repository·DTO·`LocalPasswordResetMailSender`는 만들어져 있지만 이를 연결하는 서비스·컨트롤러가 없고(`AuthController`는 signup/login/refresh/logout/me 5개뿐), 프론트엔드에도 관련 화면이 없다. `PRD.md` NF-30·NF-31, 이 문서의 `/api/auth/password/**` permitAll 설정 등 여러 문서가 이 기능의 존재를 전제하고 있다. 새 API+서비스+화면을 구현해야 하는 범위라 이번 Phase에서 만들지 않는다.
>
> **미확인 — 정상 흐름에서 도달 어려움**: `FORBIDDEN`(역할 기반 인가가 없어 트리거 경로 없음), `INTERNAL_ERROR`(서버를 살려둔 채 500을 강제해야 함, 재현 안 함).
>
> **대기 중**: 두 번째 브라우저 엔진(Edge) 확인 — PowerShell로 Edge를 열어 사용자에게 직접 확인을 요청했고 응답 대기 중이다.
>
> **자동화 아티팩트로 확정, 앱 결함 아님**: `computer.type`으로 회원가입 폼을 채워 제출했을 때 요청이 아예 안 나가고 폼이 리셋되는 것처럼 보이는 현상이 있었으나, JS로 React controlled input에 네이티브 값을 직접 주입해 재현하니 완전히 정상 동작(EMAIL_DUPLICATED 인라인 에러 정확 표시, 폼 상태 보존)했다. 이 세션의 입력 방식 문제였다.

> ### Phase 10 후속 검증 (2026-09-03)
>
> 미확정 2건을 다시 파고들어 **1건 해소 + 결함 2건 발견·수정**했다. 통과 수는 35 → 36이다.
>
> #### 에러 문구 매핑 7종 — 해소
>
> | 코드 | 화면 문구 | 확인 경로 |
> |---|---|---|
> | `INVALID_INPUT` | 입력값이 올바르지 않습니다. | 실제 서버 응답 |
> | `EMAIL_DUPLICATED` | 이미 사용 중인 이메일입니다. | 실제 서버 응답 |
> | `UNAUTHORIZED` | 이메일 또는 비밀번호가 올바르지 않습니다. | 실제 서버 응답 |
> | `NOT_FOUND` | 요청한 리소스를 찾을 수 없습니다. | 실제 서버 응답 |
> | `INTERNAL_ERROR` | 서버 오류가 발생했습니다. | **실제 서버 응답** (아래 결함으로 재현됨) |
> | `FORBIDDEN` | 접근 권한이 없습니다. | 응답 주입 — **프론트 매핑만** |
> | `RESET_TOKEN_INVALID` | 링크가 만료되었거나 이미 사용되었습니다. | 응답 주입 — **프론트 매핑만** |
> | (네트워크 실패) | 연결에 실패했습니다. | 실제 오프라인 재현 |
>
> 주입 2종은 **서버가 그 코드를 던지는 경로 자체가 없다.** `FORBIDDEN`은 `JwtAccessDeniedHandler`에만 있는데 CSRF가 비활성이고 인가 규칙이 `anyRequest().authenticated()` 하나뿐이라 `AccessDeniedException`이 발생할 수 없다(소유권 위반은 전부 404). `RESET_TOKEN_INVALID`는 비밀번호 재설정 API가 미구현이다. 두 경우 모두 `apiClient → normalize → toDisplayMessage → 화면 렌더`의 프론트 경로 전체는 실제로 통과시켰으나, **서버 동작을 확인한 것은 아니다.**
>
> #### 발견·수정한 결함 2건
>
> **1. 잘못된 요청 본문이 500으로 응답됐다** (백엔드, `fix: 잘못된 요청 본문이 500으로...`)
>
> `INTERNAL_ERROR` 재현 경로를 찾다가 발견했다. `priority: "NOT_A_PRIORITY"`나 `dueDate: "not-a-date"`를 보내면 `HttpMessageNotReadableException`이 전용 핸들러 없이 `@ExceptionHandler(Exception.class)` 폴백으로 떨어져 **500 INTERNAL_ERROR**가 나갔다. 클라이언트 오류이므로 400 INVALID_INPUT이 맞다. 500으로 응답하면 서버 로그에 ERROR로 쌓여 실제 장애와 구분되지 않는다.
>
> 같은 파일의 `MissingRequestCookieException` 핸들러에 *"이 핸들러가 없으면 아래 generic 핸들러로 떨어져 500이 나간다"*는 주석이 이미 있었다. 동일한 함정을 한 번 더 밟은 것이다.
>
> - 실측(수정 후): 잘못된 enum·날짜 → 400 `INVALID_INPUT`, 정상 요청 → 200, 빈 제목 → 400 + `details.title` 유지. `./mvnw test` 전체 통과
>
> **2. 긴 닉네임에서 헤더가 320px를 넘쳤다** (프론트, `fix: 긴 닉네임에서 헤더가...`)
>
> 반응형 항목이 `[x]`였지만 짧은 닉네임에서만 성립하고 있었다. 닉네임은 50자까지 허용되는데 축소 처리가 없었다. `min-w-0`이 없으면 flex 아이템의 최소 크기가 콘텐츠 크기로 고정돼 `truncate`만으로는 동작하지 않는다.
>
> #### 남은 1건 — 두 번째 렌더링 엔진
>
> ROADMAP이 대안으로 제시했던 **Edge는 Chromium(Blink)이라 "다른 엔진"이 아니다.** 지난 세션이 기다리던 Edge 확인은 받았더라도 이 항목을 충족시키지 못했을 것이다. 이 PC에는 Gecko·WebKit이 전혀 없어(Firefox 미설치, Playwright도 Blink) 환경상 충족 불가다. 선택지는 (a) Firefox 설치 후 확인, (b) 항목을 "단일 엔진 확인"으로 낮춰 명시적으로 인정, (c) 배포 후 실기기 확인으로 미룸 — 사용자 결정 사항이다.
>
> **결정 (2026-09-07): (c) 배포 후 실기기 확인으로 미룬다.** Phase 11 DoD에 항목을 추가했다. Phase 10은 이 1건을 이연 상태로 남긴 채 ✅로 표시하고, `v1.0.0` 태그는 Phase 11에서 이 항목까지 확인한 뒤 건다.

---

## Phase 6 이후 변경 — UX 개선 6건 (2026-09-03)

Phase 10 종료 후 사용자 요청으로 처리했다. 새 Phase가 아니라 기존 Phase의 결정을 고친 것이므로 여기에 기록한다.

### 1. 마감일 하한

`<input type="date">`에 `min`을 걸어 지난 날짜를 고를 수 없게 했다. `form`에 `noValidate`가 있어 브라우저 검증이 꺼져 있으므로 `validateDueDate`로 제출 경로도 막았다.

**단, 하한은 상황에 따라 내려간다.** 서버(`Todo.java`)에는 날짜 제약이 없어 이미 지난 마감일을 가진 데이터가 실제로 존재할 수 있고, 그것을 수정하는 것까지 막으면 제목만 고치려 해도 걸린다. 기존 값이 오늘보다 이전이면 하한을 그 값까지 내린다.

- 실측: 오늘(`2026-09-03`)이 `min`으로 걸림 / `2020-01-01` 주입 후 제출 → **POST 요청 발생하지 않음**, "마감일은 오늘 이후로 정해 주세요." 표시, `aria-invalid="true"`
- 실측(엣지): `dueDate=2020-01-01`인 항목의 수정 화면 → `min`이 `2020-01-01`로 내려가고, 제목만 고쳐 저장 시 에러 없이 성공

### 2. 할 일 추가 후 이동 경로

`/todos/{id}` → `/todos`. ~~수정 화면은 종전대로 상세에 남는다.~~ → **수정 화면도 같은 규칙으로 통일했다(같은 날 후속 요청).** 저장 후 그 화면에 남을 이유가 없다. 둘 다 `replace`라 뒤로가기가 방금 떠난 편집 화면으로 되돌아오지 않는다.

이동 전에 `setIsDirty(false)`를 먼저 호출한다. `useUnsavedChanges`가 dirty일 때 `history`에 쌓아둔 popstate 가드가 이 시점에 정리된다.

- 실측: `/todos/10130`에서 제목 수정 후 저장 → `/todos`로 이동, 바뀐 제목이 목록에 반영, 이탈 확인 다이얼로그 뜨지 않음

### 3. 다크 모드 토글 — **Phase 6 결정 번복**

`PRD.md` F-33은 원래 다크 **토글**을 요구했다. Phase 6에서 "토글이 없어 FOUC만 생긴다"는 이유로 `prefers-color-scheme` 전용으로 축소했는데, 이는 순환 논리였다(토글이 없는 이유가 토글을 만들지 않았기 때문). Phase 10 검증이 이를 놓친 이유도 명확하다 — 체크리스트가 ROADMAP 기준("OS 다크 설정에 따라 전환")이었지 PRD F-33 기준이 아니었다.

| 항목 | 내용 |
|---|---|
| 전환 신호 | `@media (prefers-color-scheme: dark)` → `html[data-theme="light\|dark"]` |
| 토글 단계 | 3단 (시스템 / 라이트 / 다크). **기본값 "시스템"이라 기존 OS 연동 동작이 보존된다** <br>※ 아래 5번에서 순환 버튼 → 드롭다운 메뉴로 바뀌었다 |
| 저장 | `localStorage["todo_theme"]`. "system"은 저장하지 않는다(값 없음 = 시스템) |
| FOUC 방지 | `app/layout.tsx` `<head>`의 동기 스크립트가 첫 페인트 전에 `data-theme` 확정 |
| 라이브러리 | 없음. `next-themes`를 쓰지 않는다(절대 규칙 11) |

**`data-theme`에는 선택이 아니라 해석된 결과만 들어간다.** "시스템"은 스크립트가 `matchMedia`로 읽어 `light`/`dark` 중 하나로 확정한다. 그래서 CSS는 상태가 둘뿐이고 같은 다크 토큰을 두 벌 유지하지 않아도 된다.

**`@custom-variant dark`가 반드시 필요하다.** shadcn의 `button`·`input`·`select`·`checkbox`가 `dark:` 유틸리티를 쓰는데, Tailwind 4 기본 `dark:`는 `prefers-color-scheme`이라 토큰만 바꾸면 그쪽이 따라오지 않는다.

`useTheme`은 `localStorage`를 `useState`로 복제하지 않고 **`useSyncExternalStore`로 구독**한다. ESLint `react-hooks/set-state-in-effect`가 effect 내 동기 `setState`를 막아 처음 작성한 방식이 거부됐고, 그것이 올바른 신호였다 — `getServerSnapshot`이 분리돼 hydration 불일치가 구조적으로 생기지 않고 다른 탭의 변경도 `storage` 이벤트로 따라온다.

`sonner`는 자기 DOM에 색을 직접 칠해 `data-theme`을 따르지 않으므로 선택값을 넘기는 `ThemeToaster`로 감쌌다.

**실측 결과**

| 확인 | 결과 |
|---|---|
| 3단 순환 | 다크 → 시스템(저장값 `null`) → 라이트 → 다크. 라벨·`data-theme`·저장값 모두 일치 |
| 새로고침 유지 | `dark` 선택 후 새로고침 → `dark` 유지, 배경 `rgb(10,10,10)` |
| FOUC | 서버 HTML에서 인라인 스크립트가 `<body>`보다 **앞**에 위치함을 확인 |
| OS 연동 보존 | "시스템" 상태 + OS 다크 + 새로고침 → `data-theme="dark"`, 배경 `rgb(10,10,10)` |
| 명시적 선택 우선 | OS 다크 + "라이트" 선택 → 새로고침해도 `light` 유지 |
| shadcn 컴포넌트 | 다크에서 input·button 배경이 함께 어두워짐 (`@custom-variant` 동작 확인) |
| hydration 경고 | 콘솔에 없음 |

> **실측하지 못한 것**: 앱 실행 **중** OS 테마를 바꿨을 때의 실시간 반영. 이 자동화 환경의 Chrome이 `prefers-color-scheme`의 `change` 이벤트를 아예 발생시키지 않았다(훅과 동일한 방식으로 단 관측용 리스너에도 오지 않았고, `matches` 값만 바뀌었다). 코드 경로 자체는 표준 `matchMedia` 구독이며, 페이지 로드 시 OS 반영은 위 표대로 실측됐다.

### 4. 할 일 추가 버튼을 화면당 하나로

빈 목록에서 헤더 버튼과 EmptyState 버튼이 **동시에** 보여 같은 동작을 하는 버튼이 둘이었다.

~~빈 상태에서는 헤더 버튼을 감추고 EmptyState 버튼만 남긴다 — 안내 문구 바로 아래에 버튼이 있어야 시선을 위로 되돌리지 않는다. 검색 결과 없음은 제외한다. 표시 조건은 `showsEmptyCta` 하나로 계산해 헤더와 렌더 분기가 함께 쓴다.~~

→ **번복됨 (같은 날 후속, 아래 6번 참고).** 버튼 개수 문제는 해결됐지만 **버튼 위치가 상태에 따라 움직이는** 새 문제를 만들었다. `showsEmptyCta`도 함께 제거됐다.

### 5. 테마 버튼을 드롭다운 메뉴로

아이콘 하나로 3단을 순환하던 방식은 현재 상태도, 다음에 무엇이 될지도 아이콘만 보고는 알기 어려웠다. 세 옵션을 이름과 함께 펼쳐 보이고 직접 고르게 한다.

- `DropdownMenuRadioGroup` 사용 → 현재 선택에 인디케이터가 붙고 `role="menuitemradio"` / `aria-checked`가 보조기술에 그대로 전달된다
- 트리거에 `ChevronDown`을 붙여 "눌리면 뭔가 열린다"는 신호를 준다. 없으면 이전 순환 버튼과 외형이 구분되지 않는다
- 옵션 정보는 배열이 아니라 `Record`다. `noUncheckedIndexedAccess`가 켜져 있어 배열 인덱스는 항상 `undefined` 가능성을 달고 다닌다

**새 의존성 없음.** 이 저장소는 개별 `@radix-ui/react-*` 대신 통합 `radix-ui`(^1.6.7) 패키지를 쓰고 `dropdown-menu`가 이미 그 안에 있다. 따라서 절대 규칙 11에 해당하지 않는다. shadcn이 생성한 `ui/dropdown-menu.tsx`는 프로젝트 Prettier 설정에 맞춰 포맷했다(기존 `ui/` 컴포넌트와 동일한 처리).

**실측 결과**

| 확인 | 결과 |
|---|---|
| 메뉴 열기 | 3개 항목이 이름·아이콘과 함께 표시, 현재 선택에 인디케이터 |
| 마우스 선택 | "다크" 클릭 → `data-theme="dark"`, 배경 `rgb(10,10,10)`, `localStorage` 저장, 트리거 아이콘 갱신 |
| 키보드 조작 | 트리거 포커스 → `Enter` 열림(`aria-expanded=true`, 포커스가 메뉴 안으로) → `Home`/`↓`로 이동 → `Enter` 선택 성공 |
| `Escape` | 닫히고 포커스가 트리거로 복귀 |
| 선택 후 포커스 | 트리거로 복귀 (측정을 너무 일찍 하면 복원 중이라 `false`로 보인다) |
| aria | `aria-haspopup="menu"`, `aria-expanded` 관리, `role="menuitemradio"` 3개, `aria-checked` 현재 값에만 `true` |
| 터치 타겟 | 트리거 54×44px (NF-22 44px 충족) |

> **발견 — 이번 변경이 원인이 아닌 기존 문제**: 헤더는 **닉네임이 길면 320px에서 넘친다.** 닉네임 `themetester`(11자, 78px)일 때 내용물 총폭이 335px로 15px 넘쳤고, `정석`(2자)으로 바꾸면 281px로 39px 여유가 생겼다. 트리거의 `ChevronDown`은 12px이라 이를 빼도 긴 닉네임에서는 여전히 넘친다 — 원인은 닉네임 길이다. 닉네임은 최대 50자(`NICKNAME_MAX_LENGTH`)까지 허용되므로 헤더에 `min-w-0`·`truncate` 등의 처리가 필요하다. ~~이번 요청 범위 밖이라 기록만 남긴다.~~ → **Phase 10 후속 검증에서 수정 완료**(위 「Phase 10 후속 검증」의 결함 2번).

### 6. 할 일 추가 버튼 위치 고정 — 4번 번복

4번에서 버튼 개수는 하나로 만들었지만, **버튼이 상태에 따라 화면을 가로질러 움직이는** 문제를 새로 만들었다. 사용자가 검색 중에 발견했다.

- 재현 실측: 할 일 0개일 때 본문 중앙 `(660, 405)` → 검색어를 입력해 결과가 없어지면 제목줄 우측 `(977, 89)` → 검색어를 지우면 다시 중앙
- 검색 디바운스가 300ms라 **타이핑을 멈춘 뒤에야** 버튼이 움직인다. 사용자가 원인을 짐작하기 어려운 형태다

헤더 버튼을 항상 두고 EmptyState의 버튼을 없앤다. 버튼이 화면당 하나라는 4번의 성질은 유지된다. 빈 화면에서 행동 수단을 잃지 않도록 안내 문구를 `"첫 번째 할 일을 추가해 보세요."` → `"위쪽 '할 일 추가' 버튼으로 첫 번째 할 일을 만들어 보세요."`로 바꿔 버튼을 가리키게 했다.

조건을 한 곳에서 계산하려고 뒀던 `showsEmptyCta`도 제거했다. 버튼이 한 자리에 고정되면서 헤더와 동기화할 대상 자체가 사라졌다.

**교훈**: "빈 상태에는 그 화면의 primary action을 둔다"는 통념을 따랐지만, 이 화면은 검색으로 **빈 상태를 드나든다.** 상태 전이가 잦은 화면에서는 위치 안정성이 근접성보다 중요하다.

- 실측(7가지 상태의 `a[href="/todos/new"]` 위치·개수): 할 일 0개 / 입력 직후(디바운스 중) / 검색 결과 없음 / 검색어 지운 뒤 / 할 일 있음 / 검색 로딩 중 / 검색 결과 있음 — **전부 `x=977, y=89`, 개수 1개로 동일**

---

## Phase 11 — AWS 배포

**저장소**: 전체 · **Docker 사용하지 않음**

> ### ⚠️ 알려진 리스크 — Amplify의 Next.js 16 지원 (2026-09-01 확인, 2026-09-07 결정)
>
> AWS 공식 문서는 이 시점에도 Amplify Hosting의 Next.js 지원 범위를 **12~15**로 명시하고 있고 **16은 목록에 없다.**
> 출처: [SSR supported features](https://docs.aws.amazon.com/amplify/latest/userguide/ssr-supported-features.html) · [Amplify support for Next.js](https://docs.aws.amazon.com/amplify/latest/userguide/ssr-amplify-support.html)
>
> **결정 (2026-09-07): 옵션 1 — 실제로 Amplify에 배포해 동작 여부를 직접 확인한다.** 미지원 목록이 곧 실패를 뜻하지는 않으므로, 다운그레이드나 호스팅 교체 전에 먼저 시도한다(실제 코드·`CLAUDE.md` 3장·`docs/guides` 5개 문서가 모두 16 기준이라 15 다운그레이드는 비용이 크다). **11-4에서 빌드·SSR이 실패하면** 그 시점에 다음을 순서대로 검토한다(변경 범위가 작은 순):
>
> 1. 실패 원인이 좁은 범위(특정 API 미지원 등)면 해당 부분만 우회
> 2. 15.x로 다운그레이드 (`CLAUDE.md` 3장·`docs/guides` 5개 문서를 함께 수정해야 한다)
> 3. 호스팅을 바꾼다 (OpenNext + SST, Vercel, EC2 자체 호스팅 등)
>
> 대안으로 넘어가는 경우 이 문서와 위에서 언급한 문서들을 함께 갱신한다.

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

### 11-1-1. 비밀번호 재설정 메일 발송 (Phase 14에서 이관)

Phase 14는 로컬 콘솔 로그 방식(`LocalPasswordResetMailSender`, `@Profile("local")`)으로 기능을 완성했다.
**운영 프로파일용 구현체는 아직 없다.** `PasswordResetMailSender` 구현 빈이 하나도 없으면
`PasswordResetService` 주입이 실패해 **운영 프로파일 기동 자체가 안 된다** — 배포 전 반드시 처리한다.

- `@Profile("prod")` SMTP 구현체 작성 (AWS SES 또는 외부 SMTP)
- `spring-boot-starter-mail` 의존성 추가 — **`CLAUDE.md` 절대 규칙 11에 따라 착수 시 승인을 받는다**
- 운영 프로파일에 SMTP 설정 추가 (자격증명은 전부 환경변수)
- 발신 도메인 확보·검증(SES는 도메인 또는 발신 주소 검증이 선행 조건이다).
  `frontend.url`은 이미 있으므로 링크 조립에 그대로 재사용한다

### 11-2. 백엔드 (EC2)

- EC2에 JDK 21 설치
- `./mvnw package`로 jar 생성 후 전송
- **systemd 서비스로 등록** (자동 재시작, 부팅 시 기동)
- 환경변수는 systemd `EnvironmentFile`로 주입 (`.env` 커밋 금지)
- EC2 보안그룹 인바운드: **80과 443을 공개**, 22는 본인 IP로 제한, 8080은 외부에 열지 않는다
  > ⚠️ **80을 닫으면 안 된다.** 11-3의 `80 → 443` 리다이렉트가 도달 불가능해지고, **certbot의 HTTP-01 챌린지도 실패해 인증서 발급 자체가 안 된다.** 80은 열되 nginx가 443으로 리다이렉트만 하도록 구성한다 (평문으로 서비스하지 않는다)

### 11-3. HTTPS (방식 확정: nginx + certbot)

개인 프로젝트 규모이므로 **EC2 한 대에 nginx 리버스 프록시 + Let's Encrypt**로 간다. ALB + ACM은 관리가 편하지만 상시 비용이 발생해 이 규모에는 과하다.

- EC2에 nginx 설치
- nginx가 80/443을 받아 `localhost:8080`(Spring Boot)으로 리버스 프록시
  - 80은 443으로 리다이렉트만 한다 (평문 응답 금지 — 11-2에서 이미 결정)
  - `proxy_set_header X-Forwarded-Proto $scheme;` · `X-Forwarded-For $proxy_add_x_forwarded_for;` · `X-Forwarded-Host $host;`를 반드시 넣는다 (아래 forward-headers 설정의 전제)
- `certbot --nginx -d api.example.com`으로 인증서 발급
  > Route 53(또는 DNS 제공자)에 `api.example.com` A 레코드가 EC2를 가리키도록 **먼저** 만들어 둬야 certbot의 HTTP-01 챌린지가 통과한다.
- **`application-prod.properties`에 `server.forward-headers-strategy=framework` 추가.** nginx가 TLS를 종료하고 백엔드에는 평문 HTTP로 전달하므로, 이 설정이 없으면 Spring이 모든 요청을 `http`로 인식한다. 영향받는 지점:
  - Google OAuth2 리다이렉트 URI가 `{baseUrl}/login/oauth2/code/google`로 조립될 때 스킴이 `http`가 되어, 구글 콘솔에 등록한 `https://...` 리다이렉트 URI와 불일치해 콜백이 실패한다
  - 프록시 구간의 스킴 인식이 꼬이면 `X-Forwarded-*` 기반의 다른 보안 판단에도 영향을 줄 수 있어 함께 켠다
- certbot 자동 갱신 확인 — 설치 시 함께 등록되는 systemd timer(`certbot.timer`)가 활성 상태인지 확인하고 `certbot renew --dry-run`으로 사전 점검

### 11-4. 프론트엔드 (Amplify)

> ⚠️ 위 "알려진 리스크" 콜아웃 참조. Next.js 16이 Amplify 공식 지원 목록(12~15) 밖이므로, **이 단계의 첫 빌드·배포 결과가 이번 Phase 전체 방향을 가른다.**

- AWS Amplify Hosting 콘솔에서 `todo-frontend` GitHub 저장소 연결 (배포 브랜치: `main`)
- 빌드 설정 — `npm ci` → `npm run build`. Amplify가 자동 감지한 빌드 스펙을 그대로 쓰되, 감지가 틀리면 `amplify.yml`을 직접 추가한다
- 환경변수 등록: `NEXT_PUBLIC_API_BASE_URL=https://api.example.com` (11-3에서 발급한 도메인)
- 커스텀 도메인 연결 (`todo.example.com`)
- **첫 빌드·SSR 동작을 직접 확인한다.**
  - 빌드 로그에서 Amplify가 Next.js 16을 SSR 컴퓨트로 인식하는지, 빌드 자체가 성공하는지 확인
  - 배포된 URL에서 서버 컴포넌트(레이아웃 등)가 정상 렌더되는지, `'use client'` 컴포넌트가 정상 하이드레이션되는지 확인
  - 실패하면 위 리스크 콜아웃의 대안 1→2→3 순서로 넘어간다. 어떤 실패였는지(빌드 실패/런타임 500/부분 기능 깨짐)를 이 문서에 기록한 뒤 진행한다

### 11-5. 도메인 전환 마무리 (백엔드 설정 갱신)

프론트·백엔드 도메인이 11-3·11-4에서 실제로 확정된 뒤에만 할 수 있다.

- Google Cloud Console → OAuth 클라이언트의 승인된 리다이렉트 URI에 `https://api.example.com/login/oauth2/code/google` 추가 (로컬용 `http://localhost:8080/...`은 유지)
- EC2 systemd `EnvironmentFile`에 운영 환경변수 채우기
  - `CORS_ALLOWED_ORIGIN=https://todo.example.com`
  - `FRONTEND_URL=https://todo.example.com` (비밀번호 재설정 링크 조립에 사용 — `CLAUDE.md` 5장)
  - `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET`
  - `AWS_S3_BUCKET` / `AWS_REGION` / `STORAGE_SIGNING_SECRET` (Phase 13에서 이미 정의된 값 재사용)
  - 11-1-1에서 만든 SMTP/SES 자격증명
- systemd 서비스 재시작 후 기동 로그에서 `spring.profiles.active=prod`로 뜨는지 확인

**DoD**

- [ ] `https://api.example.com/swagger-ui/index.html`이 200으로 열림 (인증서 유효, nginx 프록시 정상)
- [ ] `http://api.example.com`이 `https://`로 리다이렉트됨 (평문 응답 없음)
- [ ] `https://todo.example.com`에서 서버 컴포넌트·클라이언트 컴포넌트가 정상 렌더됨 (Amplify 빌드·SSR 동작 확인 — 11-4의 실질적 판정 지점)
- [ ] 운영 URL에서 회원가입 → 로그인 → Todo 생성이 실제로 성공함 (cross-site CORS·쿠키 설정 검증)
- [ ] 운영 URL에서 구글 로그인 콜백이 성공함 (`server.forward-headers-strategy` + 구글 콘솔 리다이렉트 URI 등록 검증)
- [ ] 브라우저 개발자도구에서 `refresh_token` 쿠키가 `Secure; HttpOnly; SameSite=None`으로 설정됨을 확인
- [ ] 로그아웃 후 새로고침 시 `/login`으로 이동함 (서버 측 Refresh Token 폐기 확인 — 클라이언트 삭제만으로 끝나지 않았는지)
- [ ] 비밀번호 재설정 요청 시 콘솔 로그가 아니라 실제 이메일이 발송됨 (11-1-1 SMTP/SES 구현체 검증)
- [ ] `certbot renew --dry-run` 성공 (자동 갱신 준비 확인)
- [ ] `todo-backend`·`todo-frontend` 소스 어디에도 AWS/Google 키가 하드코딩되어 있지 않음 — `grep -rn "AKIA"` 0건
- [ ] **(Phase 10에서 이관)** 운영 URL을 Chromium 계열이 아닌 다른 렌더링 엔진(Firefox 등 Gecko, 또는 모바일 Safari 등 WebKit)에서 열어 레이아웃·다크 모드·폼 동작이 동일하게 나타남을 확인 — 개발 PC에 Blink 외 엔진이 없어 Phase 10에서는 확인 불가했다

---

## Phase 12 — 로컬 이미지 첨부

**저장소**: backend + frontend + 문서 (F-46 ~ F-51, NF-32 ~ NF-34)

> 설계 정본은 `docs/appendFileImage.md`다. 이 Phase의 작업 항목은 그 문서의 요약이며,
> 충돌 시 `CLAUDE.md` > `PRD.md` > 이 문서 > `appendFileImage.md` 순으로 우선한다.

**작업**

- `application.yml` → `application.properties` 전환 (6개 파일 + `.gitignore` 11줄 + 문서 참조)
  > `.properties`는 **ISO-8859-1로 로딩**된다. 값에는 ASCII만 쓰고 한글은 주석에만 둔다.
  > Spring Boot 4.1 문서: *"By default, properties files are imported using the ISO-8859-1 charset."*
- `attachments` 테이블 + `Attachment` 엔티티 + `AttachmentRepository` (`docs/SCHEMA.md` 5장)
  > Flyway를 도입하지 않는다. local은 `ddl-auto: update`, prod는 `db/add-attachments.sql` 수동 실행.
- `StorageService` 인터페이스 + `LocalStorageService` + `StorageSignature`
  > 서명 토큰은 **`pom.xml`에 이미 있는 jjwt를 재사용**한다. `javax.crypto.Mac`으로 새로 짜지 않는다.
  > 단 `STORAGE_SIGNING_SECRET`은 `JWT_SECRET`과 **반드시 분리**한다.
- `AttachmentService` + `AttachmentController` + `SecurityConfig` permitAll 2경로
  > 로컬 업로드 URL에도 **서명 토큰을 붙인다.** 로컬이 JWT를 요구하면 프론트가 로컬/S3를 분기해야 하고,
  > S3 presigned PUT은 `Authorization` 헤더가 붙으면 서명 검증에 실패한다. 이 지점이 F-50의 성립 조건이다.
- **`HtmlSanitizer` Safelist에 `img` 추가** + `TodoService` 연결 로직 + 고아 정리 배치
  > 첨부 수집은 **반드시 `sanitize()` 이후의 HTML**을 대상으로 한다. 순서가 뒤바뀌면
  > sanitize가 제거할 태그의 첨부까지 `LINKED`로 승격되어 본문에 없는 파일이 영구 보존된다.
- **통합 테스트 9~12번 작성** (기존 1~8번 체계를 잇는다)
- 프론트: `types/attachment.ts` · `lib/attachments.ts` · `hooks/useAttachments.ts` ·
  Tiptap 이미지 노드 · **`lib/sanitize.ts`의 `ALLOWED_TAGS`에 `img` 추가** · `globals.css`
  > `uploadFile`만 **`apiClient`를 우회**해 순수 `fetch`를 쓴다. `apiClient`는 모든 요청에
  > `Authorization` 헤더와 `credentials: "include"`를 강제하는데, 그대로 S3에 보내면 서명이 깨진다.
  > 진행률은 포기하고 불확정 스피너로 간다. `fetch`는 업로드 진행률을 제공하지 않는다.

**DoD**

- [x] `./mvnw spotless:apply` 후 `./mvnw verify` 종료 코드 0 — 기존 19건 + 신규 9~12번 전부 통과
  > 최종 38건 통과(`BUILD SUCCESS`). 신규분은 `AttachmentControllerTest`(9~12번) + `StorageSignatureTest` +
  > `LocalStorageServiceTest` + `StorageProfileSwitchTest`다 (2026-09-07)
- [x] `npm run check && npm run build` 종료 코드 0 — 커밋 `72a7050` 시점에 확인 (2026-09-07)
- [x] `docs/CHECKLIST.md` 16장 16개 시나리오 전수 통과 — 16.1~16.16 전부 ☑ (2026-09-07)
- [x] 저장된 본문 HTML에 `src`가 없고 `data-attachment-id`만 있다 (F-49) — CHECKLIST 16.16
- [x] 타인 `attachmentId` 접근이 **404**다 (F-48, NF-03) — CHECKLIST 16.9 + 통합 테스트 10번
- [x] `upload/` 폴더가 `todo-project` git status에 잡히지 않는다 — CHECKLIST 16.10
- [x] `image/svg+xml` 업로드가 거부된다 (F-47) — CHECKLIST 16.13 + 통합 테스트 11번

---

## Phase 13 — S3 전환

**저장소**: backend (F-50)

> **Phase 11(AWS 배포) 완료를 전제로 한다.** 버킷·IAM이 없으면 진행할 수 없다.

**작업**

- AWS SDK v2 `s3` 의존성 추가 — BOM으로 버전 관리 (승인됨)
  > `s3-presigner`는 **별도 아티팩트가 아니다.** `S3Presigner` 클래스는 `s3` 모듈 안에 포함돼 있다.
  > 별도로 추가하면 Maven이 "`version`이 missing"이라는 에러를 낸다 — BOM의 `dependencyManagement`에
  > 그런 아티팩트 자체가 없기 때문이다. `./mvnw dependency:tree`로 실제 해석 결과를 확인하고 알아냈다.
- `config/S3Config` (`S3Client`, `S3Presigner` 조건부 빈) + `service/storage/S3StorageService`
  > 자격증명은 `DefaultCredentialsProvider`가 자동 처리한다. **코드에 분기를 두지 않는다.**
  > 로컬 시험은 환경변수, EC2는 IAM Role.
  > `S3Client.builder()...build()`와 `S3Presigner.builder()...build()`는 생성 시점에 네트워크 호출을
  > 하지 않는다(지연 연결). 그래서 실제 버킷 없이도 `@SpringBootTest`로 빈 배선만 검증할 수 있다
  > (`StorageProfileSwitchTest`).
- `application-prod.properties`에 `app.storage.type=s3` + 버킷·리전
- AWS 콘솔: 버킷 CORS(`PUT`/`GET`/`HEAD`, `AllowedHeader`에 **`Content-Type` 필수**),
  퍼블릭 액세스 차단 유지, IAM은 해당 버킷의 `PutObject`/`GetObject`/`DeleteObject`만
  > presigned PUT은 **`Content-Type`이 서명에 포함**된다. presign 시 보낸 값과 PUT 헤더 값이
  > 한 글자라도 다르면 403이다. 로컬 테스트에서는 재현되지 않고 이 Phase에서만 터진다.

**DoD**

- [x] `app.storage.type=s3`로 기동해 Phase 12의 16개 시나리오가 동일하게 통과한다
  > 실제 버킷(`todolist-dev-ojs933327`, ap-northeast-2)으로 presign → PUT → complete → GET 전체 흐름을
  > 라이브로 확인했다. 바이트 완전 일치, Content-Type 서명 일치, 소유권 404까지 정상 (2026-09-07).
  > 최초 확인 시 버킷 정책에 `s3:GetObject`를 `Principal: "*"`로 공개 허용하는 `PublicReadGetObject`
  > 정책(정적 웹사이트 호스팅 예시 정책)이 남아 있어 서명 없는 접근이 열려 있었다 — 코드 결함이 아니라
  > 버킷 설정 문제였다. 정책 삭제 후 `403 AccessDenied`로 정상 차단 확인 (2026-09-07, `docs/CHECKLIST.md`
  > 16-1.6). presigned URL은 이 정책과 무관한 별개 인증 경로라 이 수정이 위 흐름에 영향을 주지 않는다.
- [x] **`todo-frontend`에 커밋이 발생하지 않는다** (git status 클린) — F-50의 최종 수용 기준
  > Phase 12·13 커밋 완료 후 세 저장소 모두 git status 클린 확인 (2026-09-07)
- [x] `app.storage.type=local`로 되돌리면 다시 로컬 스토리지로 동작한다
  > `StorageProfileSwitchTest` + 로컬 테스트 스위트 38건이 `app.storage.type=local`로 통과 (2026-09-07)
- [x] AWS 키가 소스 어디에도 없다 (NF-05) — `grep -rn "AKIA"` 0건 확인 (2026-09-07)

---

## Phase 14 — 비밀번호 재설정

**저장소**: backend + frontend + 문서 · **관련 요구사항**: F-41 ~ F-45, NF-30, NF-31

> ### 이 Phase가 뒤늦게 생긴 이유 (2026-09-07)
>
> 비밀번호 재설정은 `PRD.md`에 F-41~F-45로 정의돼 있고 화면 표(7.5)에 흐름까지 그려져 있었으나,
> **이 문서의 「요구사항 ↔ Phase 추적표」에 행이 없었다.** 그 표는 스스로 *"여기에 행이 없는 P0는
> 구현되지 않는다"*고 경고하고 있었고, 실제로 그대로 됐다 — 엔티티·리포지토리·DTO·메일 발송 인터페이스·
> `ErrorCode.RESET_TOKEN_INVALID`·`SecurityConfig`의 permitAll 경로·`password-reset-token.expiration-minutes`
> 설정값까지 다 만들어져 있는데 **이걸 잇는 서비스와 컨트롤러만 없는** 상태로 남았다.
> Phase 10 검증에서 "`RESET_TOKEN_INVALID`를 던지는 코드가 없다"는 형태로 발견됐다.
> 이번에 추적표에 PWD-01~07 행을 먼저 넣고 이 Phase를 만들었다.

**작업**

- `service/PasswordResetService` — 요청·검증·확정 3개 흐름
  > 토큰 생성·해시는 `RefreshTokenService`의 방식(`SecureRandom` 32바이트 → Base64URL,
  > SHA-256 → hex)을 그대로 따른다. 기존 코드를 공용 유틸로 추출하지 않는다.
  > 재설정 성공 시 `refreshTokenService.revokeAllForUser(userId)`를 **재사용**한다 (F-44).
- `service/PasswordResetRateLimiter` — NF-30(이메일·IP 10분 3회). JDK만 사용하고 Phase 12에서
  만든 `SchedulingConfig`의 `@EnableScheduling`으로 만료 항목을 주기 정리한다
- `controller/PasswordResetController` — `/api/auth/password/{forgot,verify,reset}`
  > 경로는 `PRD.md` 7.5가 지정한 값이다. `SecurityConfig`의 `PERMIT_ALL_PATHS`에
  > `/api/auth/password/**`가 **이미 있으므로 보안 설정 변경이 필요 없다.**
- `domain/User`에 `changePassword(String encodedPassword)` 추가 (`@Setter` 금지 규칙)
- `ErrorCode`에 `TOO_MANY_REQUESTS`(429) 추가 + 프론트 `ErrorCode` 유니온·문구 매핑 동기화
- **통합 테스트 13번 작성** (기존 1~12번 체계를 잇는다)
- 프론트: `(auth)/forgot-password`·`(auth)/reset-password` 화면 2개 +
  `hooks/usePasswordReset.ts` + 로그인 화면에 "비밀번호를 잊으셨나요?" 링크
  > `reset-password`는 `useSearchParams`로 `?token=`을 읽으므로 `login/page.tsx`처럼
  > `<Suspense>`로 감싸야 한다.

**DoD**

- [x] `./mvnw spotless:apply` 후 `./mvnw verify` 종료 코드 0 — 기존 38건 + 신규 13번 통과
  > 실측 44건 통과 (`BUILD SUCCESS`, 2026-09-07)
- [x] `npm run check && npm run build` 종료 코드 0 (2026-09-07)
- [x] `docs/CHECKLIST.md` 17장 시나리오 전수 통과 — 14개 항목 전부 ☑ (2026-09-07)
- [x] 없는 계정·소셜 전용 계정·정상 계정의 응답과 화면 문구가 **완전히 동일**하다 (NF-31, F-45)
- [x] 같은 링크를 두 번 쓰면 두 번째가 400 `RESET_TOKEN_INVALID`다 (F-43)
- [x] 재설정 후 기존 세션이 전부 끊기고 **자동 로그인되지 않는다** (F-44)
- [x] 10분 내 4번째 요청이 429로 막힌다 (NF-30)

> ### Phase 14 검증 기록 (2026-09-07)
>
> **발견해 수정한 결함 1건 — 유효한 링크가 만료 안내로 표시됐다.** `GET .../verify`는 유효하면
> 204(본문 없음)를 주는데, `queryFn`이 그대로 `undefined`를 반환하면 React Query가 이를 오류로
> 취급한다(`Query data cannot be undefined`). 서버·네트워크는 정상이고 `npm run check`·`build`도
> 통과해서, **브라우저로 실제 링크를 열어보지 않았으면 놓쳤을 결함**이다. Phase 12에서 Tiptap 이미지
> 노드가 조용히 사라졌던 것과 같은 부류다 — 타입 검사와 빌드가 잡지 못하는 런타임 계약 문제.
>
> **테스트 격리 주의**: rate limiter는 메모리에 상태를 들고 있어 `@Transactional` 롤백으로 되돌아가지
> 않는다. MockMvc 기본 IP가 모든 요청에서 같으므로, 테스트마다 다른 클라이언트 IP를 주입해야 네 번째
> 요청부터 429가 나며 순서에 따라 깨지는 일이 없다.
>
> **`LocalPasswordResetMailSender`의 프로파일을 `local` → `!prod`로 바꿨다.** `local`로 한정하면
> 테스트 프로파일에 빈이 없어 `PasswordResetService` 주입이 실패하고 **모든 `@SpringBootTest`가**
> 컨텍스트 로딩 단계에서 무너진다.

> **운영 SMTP 발송은 이 Phase의 범위가 아니다.** 로컬은 `LocalPasswordResetMailSender`가 콘솔에
> 링크를 출력하는 방식으로 기능 전체를 완성·검증한다. 실제 메일 발송은 Phase 11로 이관했다.

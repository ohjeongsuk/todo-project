# Todo List 서비스

로컬 개발 후 AWS로 배포하는 풀스택 Todo List 서비스. 이 저장소는 **문서 저장소**이며, 실제 코드는 아래 두 저장소에 있다.

## 구조 (폴리레포)

이 프로젝트는 세 개의 **독립된 git 저장소**로 이루어져 있다. 하나로 묶인 모노레포가 아니다.

```
todo-project/          # 이 저장소. 문서만 추적한다
├── CLAUDE.md           # AI 작업 규칙의 정본
├── shrimp-rules.md      # AI 작업 세부 규칙
├── docs/
│   ├── PRD.md          # 제품 요구사항
│   ├── ROADMAP.md      # Phase 순서와 완료 판정의 정본
│   ├── SCHEMA.md        # DB 스키마 / 엔티티
│   ├── CHECKLIST.md     # 검증 체크리스트
│   └── DEV_TOOLS.md     # 개발 도구(ESLint·Prettier·Spotless 등) 설정 가이드
├── todo-backend/        # Spring Boot. 독립 git 저장소 (여기서는 추적하지 않음)
└── todo-frontend/       # Next.js. 독립 git 저장소 (여기서는 추적하지 않음)
```

`todo-backend/`·`todo-frontend/`는 이 저장소의 `.gitignore`에 통째로 제외되어 있다. 세 저장소는 각자 커밋·푸시하며, 하나의 커밋으로 묶이지 않는다.

## 시작하기

세 저장소를 각각 클론한 뒤, 아래 순서로 실행한다.

1. **백엔드** — [`todo-backend/README.md`](todo-backend/README.md) 참고 (PostgreSQL 필요)
2. **프론트엔드** — [`todo-frontend/README.md`](todo-frontend/README.md) 참고

## 기술 스택

| 영역 | 스택 |
|---|---|
| 백엔드 | Spring Boot 4, JDK 21, PostgreSQL, Spring Security |
| 프론트엔드 | Next.js 16 (App Router), React 19, TypeScript, Tailwind CSS 4 |
| 인증 | JWT(Access Token) + httpOnly 쿠키(Refresh Token), Google OAuth2 |

버전 고정 값과 전체 규칙은 [`CLAUDE.md`](CLAUDE.md)가 정본이다.

## 문서

| 문서 | 내용 |
|---|---|
| [`docs/PRD.md`](docs/PRD.md) | 제품 요구사항, 화면 목록, 알려진 한계 |
| [`docs/ROADMAP.md`](docs/ROADMAP.md) | Phase별 작업 계획과 완료 판정(DoD)의 정본 |
| [`docs/SCHEMA.md`](docs/SCHEMA.md) | DB 스키마 / 엔티티 관계 |
| [`docs/CHECKLIST.md`](docs/CHECKLIST.md) | 검증 체크리스트 |
| [`docs/DEV_TOOLS.md`](docs/DEV_TOOLS.md) | ESLint·Prettier·Spotless 등 개발 도구 설정 근거 |

## 알려진 한계

- Access Token은 서버에서 즉시 무효화할 수 없다. 로그아웃 시 Refresh Token은 즉시 폐기되지만, 이미 발급된 Access Token은 남은 유효시간(최대 30분)까지 동작한다.
- Access Token을 localStorage에 두므로 XSS가 곧 토큰 탈취로 이어진다. Tiptap 본문의 이중 정화(서버 Jsoup + 클라이언트 DOMPurify)가 사실상의 방어선이다.
- Refresh Token 재사용(탈취) 감지는 사후 대응이다 — 탈취자가 먼저 사용하면 정상 사용자가 강제 로그아웃되는 방식으로 드러난다.
- 삭제된 데이터(Soft Delete)는 보존되지만 복구할 UI는 없다.
- 로컬 환경에서는 비밀번호 재설정 메일이 실제로 발송되지 않는다 — 재설정 링크가 콘솔 로그로 출력된다. 실제 발송은 운영 SMTP 설정 후에만 동작한다.
- 세션 목록 조회나 "다른 기기 로그아웃" 화면은 없다. 비밀번호 재설정 시에만 전체 세션이 폐기된다.
- 회원 탈퇴와 프로필 수정 기능이 없다.

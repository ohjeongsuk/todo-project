# DEV_TOOLS.md — 개발 도구 가이드

이 문서는 이 저장소에 설정된 코드 품질 도구의 **사용법과 설계 근거**를 정리한다.

---

## 1. 한눈에 보기

| 영역 | 도구 | 역할 | 자동 실행 시점 |
|---|---|---|---|
| 프론트 | ESLint 9 | 코드 오류·안티패턴 검출 | 커밋 시 (staged 파일) |
| 프론트 | Prettier 3 | 코드 포맷 통일 + Tailwind 클래스 정렬 | 커밋 시 / 저장 시 |
| 프론트 | TypeScript | 타입 검증 | 커밋 시 / 빌드 시 |
| 프론트 | husky + lint-staged | 커밋 전 검사 실행 | `git commit` |
| 프론트 | commitlint | 커밋 메시지 규칙 검증 | `git commit` |
| 백엔드 | Spotless | Java 포맷 통일 | `./mvnw` 빌드 시 |
| 백엔드 | Enforcer | JDK 버전 검증 | `./mvnw` 빌드 시 |
| 공통 | .gitattributes | 개행(LF) 고정 | git 체크아웃 시 |

---

## 2. 프론트엔드 (`todo-frontend/`)

### 2.1 명령어

```bash
npm run type-check     # 타입 검사만
npm run lint           # ESLint 검사만
npm run lint:fix       # ESLint 자동 수정
npm run format         # Prettier 로 전체 포맷 적용
npm run format:check   # 포맷 위반 확인 (수정하지 않음)
npm run check          # 위 3종 한번에 (type-check + lint + format:check)
```

**커밋 전에 확인하고 싶다면 `npm run check` 하나면 된다.**

### 2.2 커밋할 때 자동으로 일어나는 일

```
git commit
   │
   ├─ [pre-commit]  lint-staged 실행
   │     ├─ *.ts, *.tsx  →  tsc --noEmit  →  eslint --fix  →  prettier --write
   │     ├─ *.js, *.mjs  →  eslint --fix  →  prettier --write
   │     └─ *.css/json/md →  prettier --write
   │     ※ 하나라도 실패하면 작업 트리를 원래 상태로 되돌리고 커밋 취소
   │
   └─ [commit-msg]  commitlint 실행
         └─ 메시지가 Conventional Commits 형식인지 검증
```

**스테이징한 파일만 검사**하므로 저장소가 커져도 커밋이 느려지지 않는다.

### 2.3 커밋 메시지 규칙

```
<타입>: <한글 설명>
```

허용 타입: `feat` `fix` `chore` `test` `docs` `refactor` `style` `perf` `build` `ci` `revert`

```bash
git commit -m "feat: 할 일 목록 페이지네이션 추가"     # ✅
git commit -m "fix: 로그아웃 후 토큰이 남는 문제 수정"   # ✅
git commit -m "할 일 추가"                            # ❌ 타입 없음
git commit -m "feat: 기능 추가."                      # ❌ 끝에 마침표
```

제목 최대 100자, 본문 한 줄 최대 200자. 한글 사용을 전제로 완화한 값이다.

### 2.4 주요 ESLint 규칙

CLAUDE.md 규칙을 기계적으로 강제하는 것들이다.

| 규칙 | 수준 | 근거 |
|---|---|---|
| `@typescript-eslint/no-explicit-any` | error | CLAUDE.md: `any` 금지 |
| `no-console` | warn (`warn`/`error`는 허용) | 디버깅 잔재 방지 |
| `@typescript-eslint/consistent-type-imports` | warn | 타입 전용 import 분리 → 번들 크기 감소 |
| `eqeqeq` | error (`null` 비교는 예외) | 암묵적 형변환 사고 방지 |
| `@typescript-eslint/no-unused-vars` | warn (`_` 접두사는 무시) | 미사용 변수 정리 |

`lint-staged`는 `--max-warnings=0`으로 실행하므로 **warn도 커밋을 막는다.** 평소 개발 중에는 경고로만 보이지만, 커밋 시점에는 error와 동등하게 취급된다.

### 2.5 TypeScript 설정에서 특히 봐야 할 것

```jsonc
"noUncheckedIndexedAccess": true   // arr[0] 의 타입이 T 가 아니라 T | undefined 가 된다
"noUnusedLocals": true             // 미사용 지역변수를 컴파일 오류로
"noUnusedParameters": true         // 미사용 매개변수를 컴파일 오류로
"noImplicitOverride": true         // 오버라이드 시 override 키워드 강제
```

`noUncheckedIndexedAccess`는 `strict: true`에 **포함되지 않는 별도 옵션**이다. 배열·객체를 인덱스로 접근할 때 "값이 없을 수 있음"을 컴파일러가 강제로 확인시켜 런타임 `undefined` 크래시를 막는다.

미사용 매개변수를 의도적으로 남겨야 하면 `_` 접두사를 붙인다.

```ts
// ❌ noUnusedParameters 위반
function handler(event: Event, context: Context) { return "ok"; }

// ✅ 의도적 무시
function handler(_event: Event, _context: Context) { return "ok"; }
```

### 2.6 Prettier 설정

```jsonc
"printWidth": 100        // 줄 길이
"semi": true             // 세미콜론 사용
"singleQuote": false     // 큰따옴표
"trailingComma": "all"   // 후행 쉼표 (diff 를 깔끔하게 만든다)
"endOfLine": "lf"        // 개행은 LF (.gitattributes 와 일치)
```

**Tailwind 클래스 자동 정렬**이 `prettier-plugin-tailwindcss`로 활성화되어 있다.

```tsx
// 저장하면 자동으로
<div className="flex flex-col flex-1 items-center bg-white px-16 py-32 dark:bg-black">
// 이렇게 공식 권장 순서로 정렬된다
<div className="flex flex-1 flex-col items-center bg-white px-16 py-32 dark:bg-black">
```

`cn()`, `cva()`, `clsx()`, `twMerge()` 안의 클래스 문자열도 정렬 대상이다.

> Tailwind CSS 4 는 설정이 `app/globals.css` 의 `@theme` 블록에 있으므로,
> 플러그인도 `tailwindConfig`(v3 방식)가 아니라 `tailwindStylesheet` 로 지정한다.

### 2.7 ESLint 와 Prettier 의 역할 분리

`eslint-config-prettier/flat` 이 `eslint.config.mjs` **배열 맨 마지막**에 있다. 이 위치가 중요하다 — flat config 는 뒤에 오는 설정이 앞을 덮어쓰기 때문이다.

- **ESLint**: "이 코드는 잘못됐다" (버그, 안티패턴)
- **Prettier**: "이 코드는 보기가 다르다" (들여쓰기, 따옴표, 줄바꿈)

이 분리가 없으면 두 도구가 같은 줄을 서로 반대 방향으로 고쳐 무한 충돌한다.

---

## 3. 백엔드 (`todo-backend/`)

### 3.1 명령어

```bash
./mvnw spotless:check    # 포맷 위반 확인
./mvnw spotless:apply    # 포맷 자동 수정
./mvnw compile           # 컴파일 (검증이 자동으로 먼저 실행됨)
./mvnw test              # 테스트
```

### 3.2 빌드할 때 자동으로 일어나는 일

```
./mvnw compile / test / package
   │
   ├─ [validate]  Enforcer   → JDK 21 인지 확인 (아니면 즉시 중단)
   ├─ [validate]  Spotless   → Java 포맷 검사 (위반 시 중단)
   └─ [compile]   javac      → 컴파일
```

**프론트엔드와 달리 git 훅이 없다.** 이 저장소(`todo-backend/.git`)에는 husky 를 설치하지 않았으므로, 포맷 검증은 커밋이 아니라 **빌드 시점**에 이루어진다.

포맷 오류로 빌드가 실패하면:

```bash
./mvnw spotless:apply    # 자동으로 고친다
./mvnw compile           # 다시 빌드
```

### 3.3 Java 포맷 규칙

google-java-format **AOSP 스타일**을 사용한다.

- 들여쓰기 **4칸 공백** (탭 아님)
- 줄 길이 100자
- import 정렬: `java/javax` → 서드파티 → `com.example` → static
- 사용하지 않는 import 자동 제거
- 줄 끝 공백 제거, 파일 끝 개행 보장

> 기본값인 `GOOGLE` 스타일은 들여쓰기가 2칸이라 Java 관례 및 Spring 생태계와 어긋나 `AOSP` 를 선택했다.

`pom.xml` 자체도 `sortPom` 으로 Maven 표준 요소 순서에 맞춰 정리된다.

---

## 4. 개행 문자 (Windows 필수)

`.gitattributes` 가 모든 텍스트 파일을 **LF** 로 고정한다.

이 설정이 없으면 Windows 에서 다음 무한 루프가 발생한다.

```
Prettier 가 LF 로 포맷
  → git 의 core.autocrlf=true 가 체크아웃 시 CRLF 로 변환
    → prettier --check 가 "포맷 위반" 판정
      → 다시 포맷 … (반복)
```

`.husky/` 아래 훅 파일은 `sh` 로 실행되므로 CRLF 면 아예 동작하지 않는다. 그래서 별도 규칙으로 한 번 더 LF 를 못박았다.

`.bat` / `.cmd` 만 Windows 규약대로 CRLF 를 유지한다.

---

## 5. VS Code

`.vscode/settings.json` 에 저장 시 자동 포맷이 설정되어 있다.

- 저장하면 Prettier 포맷 + ESLint 자동 수정
- import 는 `@/` 별칭으로 자동 생성
- 프로젝트에 설치된 TypeScript 버전 사용

`.vscode/extensions.json` 의 권장 확장을 설치해야 동작한다.

| 확장 | 용도 |
|---|---|
| `esbenp.prettier-vscode` | Prettier 포맷 |
| `dbaeumer.vscode-eslint` | ESLint 실시간 검사 |
| `bradlc.vscode-tailwindcss` | Tailwind 클래스 자동완성 |
| `editorconfig.editorconfig` | 에디터 설정 통일 |

VS Code 가 우측 하단에 "권장 확장을 설치하시겠습니까?" 를 띄우면 설치하면 된다.

---

## 6. 문제 해결

### 커밋이 막혔을 때

에러 메시지에 어떤 파일의 몇 번째 줄이 문제인지 나온다. 고친 뒤 다시 `git add` 하고 커밋한다.

`lint-staged` 는 실패 시 **작업 트리를 원래대로 되돌리므로**, 파일이 반쯤 수정된 채 남는 일은 없다.

### 급하게 훅을 건너뛰어야 할 때

```bash
git commit --no-verify -m "fix: 긴급 수정"
```

**상시로 쓰면 도구를 설정한 의미가 없다.** 배포 장애 대응 같은 예외 상황에만 쓴다.

### `release version 21 not supported`

`JAVA_HOME` 이 JDK 21 이 아니다. 확인:

```bash
echo $JAVA_HOME
java -version
```

일회성으로 넘기려면:

```bash
JAVA_HOME="C:/SpringBootProject/zulu21" ./mvnw compile
```

근본 해결은 시스템 환경변수 `JAVA_HOME` 을 JDK 21 경로로 바꾸는 것이다.

### Maven 출력의 한글이 깨질 때

`.mvn/jvm.config` 가 UTF-8 을 지정하고 있다. 그래도 깨지면 터미널 자체의 인코딩 문제다.

```powershell
chcp 65001    # PowerShell / cmd 를 UTF-8 로
```

### Prettier 와 ESLint 가 서로 다르게 고칠 때

`eslint.config.mjs` 에서 `prettierConfig` 가 배열 **맨 마지막**에 있는지 확인한다. 순서가 바뀌면 충돌 해제가 동작하지 않는다.

# Tiptap 본문 이미지 첨부 (로컬 → S3)

> `docs/tiptap-s3-image-upload-prompt.md`를 실제 코드베이스 조사 결과로 재설계한 작업 지시서다.
> 원문과 충돌하면 **이 문서가 우선한다.** `CLAUDE.md`와 충돌하면 `CLAUDE.md`가 우선한다.

---

## 0. 원문 대비 변경 요약

원문 10개 장 전부에 판정을 붙였다. 근거는 각 장 본문에 있다.

### 삭제한 지시

| 원문 위치 | 지시 | 삭제 사유 |
|---|---|---|
| 0장 5번, 10장 2단계 | `local`/`prod` 프로파일을 새로 만든다 | **이미 존재한다.** `application-local.yml` / `application-prod.yml`이 `ddl-auto`까지 분리돼 있다 |
| 1장 | Flyway 도입, `V{n}__add_attachment.sql` | 이 저장소 관행이 아니다. 스키마 예외는 `db/*.sql` 수동 스크립트로 관리한다 (`db/add-title-index.sql` 선례) |
| 5장 | `server.tomcat.max-swallow-size=6MB` | **오해를 부르는 설정이다.** 이 값은 중단된 요청의 잔여 본문을 서버가 버리는 양이지 업로드 상한이 아니다 |
| 4장 | 두 저장소의 `.gitignore`에 `../upload/` 추가 | **gitignore 패턴은 상위 경로를 참조할 수 없다.** 동작하지 않는다 |
| 6장, 8장 | 소유권 불일치 시 **403** | `CLAUDE.md` 절대 규칙 4 위반. 이 프로젝트는 존재 노출 방지를 위해 **404**로 통일한다 |

### 정정한 사실

| 원문 | 실제 |
|---|---|
| 본문 컬럼 `description` | **`content`** (`@Column(columnDefinition = "TEXT")`, DTO 제약 `@Size(max = 50000)`) |
| `User`(또는 Member) 엔티티 | `User`. 컨트롤러는 `@AuthenticationPrincipal User principal`로 직접 받는다 |
| `domain/attachment/`, `storage/` 패키지 | 이 저장소는 **계층형** 구조다 (`controller`/`domain`/`dto`/`exception`/`security`/`service`) |
| 테이블명 `attachment` | **`attachments`** (기존 `users`, `todos` 복수형 관행) |
| `GET /api/attachments/{id}/url` 단건 조회 | **`POST /api/attachments/view-urls` 일괄 조회.** 본문에 이미지 5장이면 원문 설계는 요청이 5번 나간다 |

### 추가한 항목 (원문 누락)

| # | 항목 | 누락 시 증상 |
|---|---|---|
| ① | **Jsoup·DOMPurify 허용 목록에 `img` 추가** | 이미지가 저장 즉시 사라진다. 빌드·테스트는 통과한다 |
| ② | **로컬 업로드 URL에도 서명 토큰** | 프론트가 로컬/S3를 분기해야 한다. 원문의 핵심 목표가 무너진다 |
| ③ | **프론트는 `apiClient`를 우회한다** | S3 presigned PUT에 `Authorization` 헤더가 붙어 403이 난다 |
| ④ | **상태값 2개 → 3개** (`PENDING`/`UPLOADED`/`LINKED`) | "파일을 안 올린 레코드"와 "올렸지만 미저장인 레코드"를 구분 못 한다 |
| ⑤ | **업로드 크기 제한을 애플리케이션에서 구현** | 스트리밍 PUT은 Tomcat 설정으로 제한되지 않는다 |
| ⑥ | **sanitize 이후에 첨부를 수집** | sanitize가 지울 태그의 첨부까지 `LINKED`로 승격돼 영구 보존된다 |
| ⑦ | **`.properties`는 ISO-8859-1로 읽힌다** | 설정 파일의 한글 값이 깨진다 |
| ⑧ | **정본 문서 개정 단계** | `PRD.md`·`CLAUDE.md`가 이 기능을 "범위 밖"으로 명시하고 있어 계속 충돌한다 |
| ⑨ | **신규 통합 테스트 9~12번** | 원문에 테스트 작성 지침이 아예 없다 |
| ⑩ | **Spotless 통과 필수** | 포맷 불일치 시 `validate` 단계에서 빌드가 깨진다 |
| ⑪ | **진행률은 `fetch`로 불가능** | 원문 7장 2번이 구현 불가능한 요구다 |

### 유지한 지시

`StorageService` 인터페이스 추상화, `@ConditionalOnProperty` 빈 분기, presign → PUT → complete 3단 흐름,
`todo-project/upload` 저장 위치, 경로 조작 방어, 자격증명 원칙, 임시 레코드 선생성 발상, 고아 파일 정리 배치,
S3 버킷 비공개 유지, 단계별 승인 진행. **원문의 골격은 타당하다.**

---

## 1. 작업 목표

할일 작성/수정 시 Tiptap 본문에 **이미지를 첨부**할 수 있게 한다.
파일 실체는 스토리지(로컬 디렉터리 또는 S3)에, 메타데이터는 PostgreSQL에 둔다.

**로컬 파일 저장으로 먼저 완성해 검증한 뒤 S3로 전환한다.**
프론트엔드 코드는 두 방식에서 **동일해야 한다.** 스토리지 교체는 백엔드 설정만 바꿔서 이뤄진다.

`todo-project` / `todo-backend` / `todo-frontend`는 **각각 독립된 git 저장소**다. 커밋은 저장소별로 나눈다.

---

## 2. 확정된 사실

원문 0장은 "확인한 뒤 보고하라"는 지시였다. 조사가 끝났으므로 사실 명세로 대체한다.
**아래는 다시 확인할 필요가 없다.**

### 백엔드

- Spring Boot **4.1.1**, Java 21, Maven(`mvnw`). PostgreSQL 17.
- 패키지 `com.example.todoapp`, **계층형 구조**:
  `config` / `controller` / `domain` / `dto` / `exception` / `security` / `service`
- `BaseEntity`(`@MappedSuperclass`)가 `createdAt` / `updatedAt` / `deletedAt`와 `softDelete()`를 제공한다.
  `@SQLRestriction`·`@Where`는 **쓰지 않는다.** 리포지토리 쿼리마다 `...DeletedAtIsNull`을 명시한다.
- `Todo.content`가 본문 HTML이다. `@Column(columnDefinition = "TEXT")`.
- 스키마 관리: **Flyway 없음.** local `ddl-auto: update`, prod `validate`.
  ddl-auto가 표현 못 하는 것은 `db/*.sql` 수동 스크립트 (`db/add-title-index.sql` 참고).
- 에러 응답: `ApiResponse<T>(success, data, error)` + `ErrorCode` enum + `ApiException` 단일 예외.
  조립은 `GlobalExceptionHandler` 한 곳에서만.
- **소유권 실패는 404다.** `TodoService.getOwned()`가 `ErrorCode.NOT_FOUND`를 던진다.
- 인증: `JwtAuthenticationFilter`가 `User` 엔티티를 principal로 SecurityContext에 넣는다.
  컨트롤러는 `@AuthenticationPrincipal User principal`.
- CORS: `allowedOrigins`는 단일 값 주입, `allowedMethods`에 PUT 포함,
  `allowedHeaders = Authorization, Content-Type`, `allowCredentials(true)`.
- **Spotless**(google-java-format AOSP)가 `validate` 단계에 바인딩돼 있다.
- **`@EnableScheduling`이 없다.** 배치를 넣으려면 새로 추가해야 한다.
- 테스트 6개. `@SpringBootTest + @AutoConfigureMockMvc + @ActiveProfiles("test") + @Transactional`,
  `JwtTokenProvider.createAccessToken(userId)`로 **실제 JWT를 발급**해 헤더에 싣는다. `@WithMockUser` 미사용.
  통합 테스트 번호 체계는 **1~8번**이 사용 중이다.

### 프론트엔드

- Next.js 16.3.3(App Router), React 19, TypeScript strict, Tailwind 4, TanStack Query v5.
- Tiptap **3.30.6**. `src/components/todo/TodoEditor.tsx`.
  StarterKit에서 `heading`/`blockquote`/`code`/`codeBlock`/`strike`/`underline`/`horizontalRule` 비활성.
  현재 툴바: 굵게 / 기울임 / 글머리 목록 / 번호 목록 / 링크.
- **`@tiptap/extension-image` 미설치.** StarterKit에 이미지 노드가 없다.
- `src/lib/apiClient.ts`가 단일 진입점. **모든 요청에 `Authorization` 헤더 + `credentials: "include"`를 강제**하고,
  401이면 refresh를 1회 시도한다.
- `src/lib/`은 **평면 구조**다. `src/lib/api/` 디렉터리는 없다.
- 훅은 `src/hooks/`, 쿼리 키는 `src/lib/queryKeys.ts`.
- 이미지·파일 업로드 관련 코드는 **전혀 없다.**
- 검증 명령: `npm run check`(type-check + lint + format:check), `npm run build`.

### 현재 허용 HTML 태그 (이번 작업의 출발점)

```
p  strong  em  ul  ol  li  br  a
```

**세 곳에 중복 정의돼 있다.**

1. `todo-backend/src/main/java/com/example/todoapp/service/HtmlSanitizer.java` — Jsoup Safelist (정본)
2. `todo-frontend/src/lib/sanitize.ts` — DOMPurify `ALLOWED_TAGS`
3. `todo-frontend/src/components/todo/TodoEditor.tsx` — Tiptap 확장 설정

**하나라도 빠뜨리면 "넣었는데 저장하면 사라지는" 증상이 재발한다.**

---

## 3. 0단계 — 정본 문서 개정

`PRD.md` 3장과 `CLAUDE.md` 2장이 **"파일 첨부 / 이미지 업로드는 범위 밖, 따라서 Amazon S3도 스택에서 제외"**
라고 명시하고 있다. 이 상태로 구현하면 모든 후속 작업이 정본 문서 위반이 된다. **구현보다 먼저 고친다.**

| 문서 | 변경 |
|---|---|
| `docs/PRD.md` | 3장 "제외" 표에서 `파일 첨부 / 이미지 업로드` 행 삭제. 4.3에 **F-46 ~ F-51** 신설(이미지 첨부 / 업로드 제한 / 소유권 / 고아 정리 / 스토리지 추상화 / S3 전환). 5.1 보안에 **NF-32 ~ NF-34**(서명 URL 만료, 경로 조작 방어, SVG 금지) |
| `CLAUDE.md` (루트·`todo-project` 양쪽) | 2장 인프라의 S3 제외 문장을 사용 조건 기술로 교체. 절대 규칙 8에 `img[data-attachment-id]` 취급 추가. 절대 규칙 9의 `application-local.yml` → `application-local.properties`. 4장 Java 컨벤션의 "기능별 패키지" 서술을 실제 계층형 구조로 정정 |
| `shrimp-rules.md` | 7장 금지 목록의 **"❌ Amazon S3 / 파일 업로드 기능 추가"** 삭제 — 이 조항이 남으면 이후 모든 구현이 규칙 위반이 된다. 함께 정정할 오류 5건: 4.2·7장의 `src/` 금지(실제가 `src/` 구조), 4.4의 다크모드 `.dark` 클래스(실제는 `data-theme`), 4.4의 shadcn `radix-nova`(실제는 `new-york`), 3.3의 기능별 패키지(실제는 계층형), 9장 전환 중 항목(대부분 해소됨) |
| `docs/ROADMAP.md` | **Phase 12 — 로컬 이미지 첨부**, **Phase 13 — S3 전환**(Phase 11 배포 완료 전제) 신설. 진행 현황 표 갱신. 요구사항↔Phase 추적표에 F-46~51 / NF-32~34 추가. 통합 테스트 **9~12번** 배정 |
| `docs/SCHEMA.md` | **5장 `attachments` 테이블** 신설. 기존 5열 표 포맷(`컬럼 / 타입 / NULL / 제약 / 설명`) 준수 |
| `docs/CHECKLIST.md` | **16장(이미지 첨부)**, **16-1(스토리지 전환)** 신설 |

> `docs/SCHEMA.md`가 엔티티 경로를 `com.example.domain.Todo`로 적고 있으나 실제는
> `com.example.todoapp.domain.Todo`다. 기존 문서 표류이며, 5장을 추가하는 김에 함께 고친다.

---

## 4. 1단계 — `application.yml` → `application.properties` 전환

원문은 이것을 확인 항목 하나로 묻어뒀으나, 실제로는 **연쇄 수정이 큰 독립 작업**이다.
다른 작업과 섞지 말고 이 단계에서 끝낸 뒤 커밋한다.

### 전환 대상 6개

```
src/main/resources/application.yml               → application.properties
src/main/resources/application-local.yml         → application-local.properties        (비추적)
src/main/resources/application-prod.yml          → application-prod.properties
src/main/resources/application-local.yml.example → application-local.properties.example
src/test/resources/application-test.yml          → application-test.properties          (비추적)
src/test/resources/application-test.yml.example  → application-test.properties.example
```

### 함께 고쳐야 하는 곳

- `todo-backend/.gitignore` — "로컬 설정 및 비밀 정보" 절의 `application-local.yml` / `.yaml` /
  `application-secret.yml` / `application-test.yml`과 대응하는 `!...example` 예외 **11줄을 `.properties` 기준으로 교체**
- 루트 `CLAUDE.md`, `todo-project/CLAUDE.md` — 절대 규칙 9의 파일명
- `docs/DEV_TOOLS.md`, `docs/ROADMAP.md`, `docs/CHECKLIST.md` — `application-*.yml` 참조 전수 교체

### 반드시 지킬 제약 — ISO-8859-1

Spring Boot 4.1 공식 문서(External Configuration):

> *"By default, properties files are imported using the ISO-8859-1 charset."*

`[encoding=utf-8]` 속성은 `spring.config.import`에만 있고 **표준 `application-{profile}.properties`
로딩에는 적용되지 않는다.** YAML에는 없던 제약이다.

- **값에는 ASCII만 쓴다.** 한글은 주석에만 (주석은 파서가 버리므로 런타임 무해)
- 현재 `application-local.yml.example`의 한글 플레이스홀더(`여기에_로컬_DB_비밀번호를_입력한다` 등)를
  **ASCII로 교체한다.** 전환 시 가장 먼저 깨질 지점이다
- YAML 중첩 키는 점 표기로 평탄화한다. `spring.jpa.properties.hibernate.format_sql`처럼 접두사 누락에 주의
- 프로파일 활성화는 파일명 규약 그대로 동작하므로 `spring.config.activate.on-profile`은 필요 없다

### DoD

```bash
./mvnw spotless:apply test
```

기존 테스트 19건이 모두 통과해야 한다. 전환만으로 동작이 달라지면 안 된다.

---

## 5. DB 스키마

새 테이블 **`attachments`** (복수형).

| 컬럼 | 타입 | NULL | 제약 | 설명 |
|---|---|---|---|---|
| `id` | `bigint` | NOT NULL | PK, IDENTITY | |
| `todo_id` | `bigint` | NULL | FK → `todos.id` | **NULL 허용** — 아래 사유 참고 |
| `user_id` | `bigint` | NOT NULL | FK → `users.id` | 업로더. 소유권 검증의 기준 |
| `storage_type` | `varchar(20)` | NOT NULL | CHECK (`LOCAL`,`S3`) | `StorageType` enum |
| `storage_key` | `varchar(512)` | NOT NULL | UNIQUE | 로컬 상대경로 또는 S3 객체 키. **서버만 생성한다** |
| `original_filename` | `varchar(255)` | NOT NULL | | 원본 파일명 |
| `content_type` | `varchar(100)` | NOT NULL | | 화이트리스트 검증을 통과한 값 |
| `file_size` | `bigint` | NOT NULL | | bytes. `complete` 시점에 실측값으로 덮어쓴다 |
| `status` | `varchar(20)` | NOT NULL | CHECK (`PENDING`,`UPLOADED`,`LINKED`) | 아래 참고 |
| `created_at` / `updated_at` / `deleted_at` | | | | `BaseEntity` 공통 필드 |

**인덱스**

- `idx_attachments_todo_deleted` — `(todo_id, deleted_at)` — 할일 상세 조회
- `idx_attachments_user_deleted` — `(user_id, deleted_at)` — 소유권 검증
- `idx_attachments_status_created` — `(status, created_at)` — 고아 정리 배치

**저장 키 규칙**: `todos/{userId}/{yyyy}/{MM}/{uuid}.{ext}` (로컬·S3 동일)

### `todo_id`를 NULL 허용으로 두는 이유

에디터에서는 할일을 저장하기 *전에* 이미지가 먼저 업로드된다. 업로드 시점에는 연결할 할일이 아직 없다.

### 상태값을 3개로 나누는 이유 (원문은 2개)

| 상태 | 의미 | 전이 시점 |
|---|---|---|
| `PENDING` | 레코드만 있고 **파일이 없다** | `presign` 응답 시 |
| `UPLOADED` | 파일은 확정됐으나 **아직 할일에 연결 안 됨** | `complete` 성공 시 |
| `LINKED` | 할일 본문에 실제로 존재한다 | 할일 저장/수정 시 |

원문의 `TEMP` / `LINKED` 2개로는 "presign만 하고 파일을 안 올린 레코드"와
"파일은 올렸지만 사용자가 할일을 저장하지 않은 레코드"를 구분할 수 없다.
두 경우는 정리 정책이 달라야 한다 (10장 참고).

### `storage_type`을 두는 이유

로컬에서 테스트한 데이터와 S3 전환 이후 데이터가 섞여도 각각 올바른 방식으로 조회하기 위함이다.

### 마이그레이션

**Flyway를 도입하지 않는다.** 이 저장소 관행을 따른다.

- local: `ddl-auto: update`가 엔티티에서 테이블을 만든다
- prod: `ddl-auto: validate`이므로 **`db/add-attachments.sql`을 수동 실행**한다
- 스크립트 형식은 `db/add-title-index.sql`을 따른다 — 주석에 사유와 psql 실행 명령을 적고,
  `todolist_db`와 `todolist_db_test` **양쪽에 실행**한다

---

## 6. 스토리지 추상화

로컬과 S3를 같은 인터페이스로 다룬다. `AttachmentService`는 어떤 구현체인지 알지 못한다.

```java
public interface StorageService {

    StorageType getType();

    /** 업로드용 URL. S3는 presigned PUT, 로컬은 서명 토큰이 붙은 백엔드 엔드포인트. */
    String createUploadUrl(Long attachmentId, String storageKey, String contentType);

    /** 조회용 URL. S3는 presigned GET, 로컬은 서명 토큰이 붙은 백엔드 엔드포인트. */
    String createViewUrl(Long attachmentId, String storageKey);

    /** 업로드 완료 후 실제 파일의 존재와 크기를 확인해 실측 바이트를 반환한다. */
    long verifyUploaded(String storageKey);

    void delete(String storageKey);
}
```

> 원문 시그니처에 `attachmentId`를 추가했다. 로컬 구현이 서명 토큰을 만들려면 대상 식별자가 필요하다.

구현체는 `app.storage.type` 값에 따라 `@ConditionalOnProperty`로 **하나만** 등록한다.

- `service/storage/LocalStorageService` — `havingValue = "local"`
- `service/storage/S3StorageService` — `havingValue = "s3"`

---

## 7. 업로드 흐름 (로컬·S3 공통)

```
1. 프론트 → 백엔드    POST /api/attachments/presign  { filename, contentType, fileSize }
2. 백엔드             검증 → attachments 레코드 생성(status=PENDING) → 업로드 URL 발급
                      ← { attachmentId, uploadUrl }
3. 프론트 → uploadUrl PUT (파일 본문, Content-Type 헤더 포함)
4. 프론트 → 백엔드    POST /api/attachments/{id}/complete
5. 백엔드             실제 파일 존재·크기 확인 → status=UPLOADED
                      ← { attachmentId, viewUrl }
```

### `uploadUrl` / `viewUrl`이 가리키는 곳

| | uploadUrl | viewUrl |
|---|---|---|
| 로컬 | `http://localhost:8080/api/attachments/{id}/upload?token=...` | `http://localhost:8080/api/attachments/{id}/raw?token=...` |
| S3 | S3 presigned PUT URL | S3 presigned GET URL (유효기간 30분) |

### 응답에서 `storageKey`를 뺀다 (원문은 포함)

클라이언트는 `attachmentId`만으로 모든 작업을 한다.
`storageKey`를 내보내지 않으면 **클라이언트가 경로를 되돌려보낼 통로 자체가 없어져**,
경로 조작 공격면이 원천적으로 사라진다. 서버 내부의 심층 방어(8장)는 그대로 유지한다.

### ② 로컬 업로드 URL에도 서명 토큰을 붙인다 — 원문 누락

원문은 조회(`/raw`)에만 서명 토큰을 언급했다. **업로드에도 필요하다.**

- 로컬 `PUT .../upload`가 JWT를 요구하면 → 프론트가 `Authorization` 헤더를 붙여야 한다
- S3 presigned PUT에 `Authorization` 헤더나 쿠키를 보내면 → **서명 검증 실패(403)**
- 결과: 프론트가 로컬/S3를 분기하게 되고, **"프론트 코드는 동일하다"는 이 작업의 핵심 목표가 무너진다**

**해결**: 로컬 `uploadUrl`도 단기 서명 토큰을 쿼리로 붙여 발급한다.
`SecurityConfig`의 permitAll 목록에 두 경로를 추가하고, **토큰 자체로 인증**한다.

```
/api/attachments/*/upload
/api/attachments/*/raw
```

토큰은 `attachmentId` + 용도(upload/view) + 만료시각을 서명한 값이다.
`JWT_SECRET`을 재사용하지 말고 별도 키를 쓴다. 만료는 `app.storage.url-expiry-minutes`(기본 30분).

S3 버킷은 퍼블릭으로 열지 않는다.

---

## 8. 로컬 스토리지 구현

### 저장 위치

`todo-project/upload`를 사용한다. `todo-backend`·`todo-frontend`와 형제 레벨이며 **어느 저장소에도 커밋하지 않는다.**

```
todo-project/
├── todo-backend/
├── todo-frontend/
└── upload/          ← 여기
```

- 경로는 설정값으로 주입한다 (하드코딩 금지)
- 애플리케이션 시작 시 디렉터리가 없으면 생성한다
- **`.gitignore`는 `todo-project/.gitignore`에만 `upload/`를 추가한다**

> 원문의 "두 저장소의 `.gitignore`에 `upload/`, `../upload/`를 추가한다"는 **동작하지 않는다.**
> gitignore 패턴은 상위 경로(`../`)를 참조할 수 없고, `upload/`는 `todo-backend`·`todo-frontend`
> 저장소 **바깥**이라 그쪽 `.gitignore`와는 애초에 무관하다.

### 경로 조작 방어

`storageKey`는 서버만 생성하고 응답에도 넣지 않으므로 클라이언트가 제어할 수 없다.
그래도 심층 방어로 남긴다 — 저장·조회 전에 최종 경로를 `Path.normalize()`로 정규화한 뒤
**base 디렉터리 하위인지 확인**하고, 벗어나면 예외를 던진다.

### 업로드 엔드포인트 `PUT /api/attachments/{id}/upload?token=`

- 서명 토큰 검증 → `status`가 `PENDING`인지 확인 → 이미 파일이 있으면 **409**
- 요청 본문을 **스트림으로** 파일에 기록한다. 메모리 전체 적재 금지

#### ⑤ 크기 제한은 애플리케이션에서 한다 — 원문의 설정은 무력하다

`server.tomcat.max-swallow-size`는 **중단된 요청의 잔여 본문을 서버가 버리는 양**이지 업로드 상한이 아니다.
`maxPostSize`도 form 콘텐츠 타입에만 적용되므로 스트리밍 PUT에는 걸리지 않는다.

이중 방어로 구현한다.

1. `Content-Length` 헤더를 상한과 먼저 비교해 즉시 거절 (빠른 실패)
2. 스트림을 읽으며 바이트를 세고, 상한 초과 시 **중단 + 부분 파일 삭제**
   (`Content-Length`는 위조 가능하므로 1번만으로는 부족하다)

### 조회 엔드포인트 `GET /api/attachments/{id}/raw?token=`

- 서명 토큰 검증 후 `Content-Type`과 함께 파일 스트림 반환
- `Content-Disposition: inline`
- `X-Content-Type-Options: nosniff` — 브라우저의 MIME 스니핑 차단
- 브라우저 `<img>`가 직접 호출하므로 JWT 헤더를 실을 수 없다. **그래서 서명 토큰 방식이다.**
  S3 presigned GET과 개념이 같아 전환 시 구조가 바뀌지 않는다

---

## 9. 백엔드 설정 (`.properties`)

> 값에는 ASCII만 쓴다 (4장 ISO-8859-1 제약).

### `application.properties` (공통)

```properties
# 파일 업로드 제한
app.upload.max-file-size=5242880
app.upload.allowed-content-types=image/jpeg,image/png,image/gif,image/webp
```

`image/svg+xml`은 **넣지 않는다.** SVG는 스크립트를 실행할 수 있어 XSS 벡터다.

### `application-local.properties`

```properties
app.storage.type=local
app.storage.local.base-dir=${LOCAL_UPLOAD_DIR:../upload}
app.storage.local.base-url=http://localhost:8080
app.storage.url-expiry-minutes=30
app.storage.signing-secret=${STORAGE_SIGNING_SECRET}
```

### `application-prod.properties`

```properties
app.storage.type=s3
app.storage.s3.bucket=${AWS_S3_BUCKET}
app.storage.s3.region=${AWS_REGION:ap-northeast-2}
app.storage.url-expiry-minutes=30
app.storage.signing-secret=${STORAGE_SIGNING_SECRET}
```

### 자격증명 원칙

**AWS 키를 코드나 설정 파일에 절대 넣지 않는다.**

- 로컬에서 S3를 시험할 때만 환경변수 `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` 사용
- EC2 배포 시에는 IAM Role (키 없음)
- 둘 다 `DefaultCredentialsProvider`가 자동 처리하므로 **코드에 분기를 두지 않는다**
- `application-prod.properties`, `.env`가 `.gitignore`에 있는지 확인한다

---

## 10. 백엔드 구현 목록

### 파일 배치 — 계층형 구조 준수

```
domain/Attachment.java                 엔티티. BaseEntity 상속
domain/AttachmentStatus.java           PENDING / UPLOADED / LINKED
domain/StorageType.java                LOCAL / S3
domain/AttachmentRepository.java       ...DeletedAtIsNull 조건 명시
dto/AttachmentPresignRequest.java      record
dto/AttachmentPresignResponse.java     record — storageKey 미포함
dto/AttachmentResponse.java            record
dto/AttachmentViewUrlsRequest.java     record — 일괄 조회
service/AttachmentService.java         검증 / 상태 전이 / 소유권
service/AttachmentCleanupScheduler.java
service/storage/StorageService.java
service/storage/LocalStorageService.java
service/storage/S3StorageService.java
service/storage/StorageSignature.java  업로드·조회 공통 서명 토큰 발급·검증
config/S3Config.java                   S3Client / S3Presigner (조건부)
config/SchedulingConfig.java           @EnableScheduling — 현재 저장소에 없다
controller/AttachmentController.java
```

### API

| 메서드 | 경로 | 인증 | 비고 |
|---|---|---|---|
| POST | `/api/attachments/presign` | JWT | 업로드 URL 발급 |
| POST | `/api/attachments/{id}/complete` | JWT | 업로드 확정 |
| POST | `/api/attachments/view-urls` | JWT | **일괄 조회** |
| DELETE | `/api/attachments/{id}` | JWT | Soft Delete |
| PUT | `/api/attachments/{id}/upload?token=` | 서명 토큰 | 로컬 전용, permitAll |
| GET | `/api/attachments/{id}/raw?token=` | 서명 토큰 | 로컬 전용, permitAll |

원문의 `GET /api/attachments/{id}/url`(단건)을 **일괄 조회로 교체**했다.
본문에 이미지가 5장이면 단건 방식은 요청이 5번 나간다.

### 규칙

- **소유권 불일치는 404** (`ErrorCode.NOT_FOUND`). 403이 아니다 — `CLAUDE.md` 절대 규칙 4, PRD NF-02
- `ErrorCode`에 추가: `FILE_TOO_LARGE`(400), `UNSUPPORTED_FILE_TYPE`(400), `UPLOAD_NOT_COMPLETED`(400)
- 모든 응답은 `ApiResponse<T>` 래핑. 예외 조립은 `GlobalExceptionHandler`에서만
- DTO는 **record**, 엔티티는 클래스. `@Setter` 금지 →
  `markUploaded(actualSize)`, `linkTo(todo)`, `softDelete()` 같은 의미 있는 메서드
- 생성자 주입만. 조회 메서드에 `@Transactional(readOnly = true)`
- `contentType`은 **화이트리스트로 검증한다.** 확장자만 믿지 않는다
- 커밋 전 `./mvnw spotless:apply` — 미실행 시 `validate` 단계에서 빌드가 깨진다

### ① Jsoup Safelist에 `img` 추가 — 이 작업의 성패를 가르는 지점

`service/HtmlSanitizer.java`:

```java
.addTags("p", "strong", "em", "ul", "ol", "li", "br", "a", "img")
.addAttributes("img", "alt", "data-attachment-id")
```

**`src`는 허용하지 않는다.** 조회 URL은 만료되므로 본문 HTML에 박아두면 며칠 뒤 전부 깨진다.
`data-attachment-id`만 저장하고 렌더 시점에 `src`를 주입한다.

부수 효과가 크다.

- `javascript:` src 주입 경로가 **원천 차단**된다
- 본문 길이가 짧아져 `@Size(max = 50000)` 제약이 안전해진다
- 스토리지를 바꿔도 이미 저장된 HTML을 손댈 필요가 없다

### ⑥ 할일 저장 시 연결 처리 — 순서가 중요하다

`TodoService.create` / `update`는 `htmlSanitizer.sanitize()`를 엔티티 조작 **직전**에 호출한다.
첨부 수집은 **반드시 그 이후**의 HTML을 대상으로 한다.

순서가 뒤바뀌면 sanitize가 제거할 태그의 첨부까지 `LINKED`로 승격되어,
본문에 존재하지 않는 파일이 영구 보존된다.

1. `content`를 sanitize한다
2. **sanitize된** HTML을 파싱해 `data-attachment-id`를 모두 수집한다
3. 해당 첨부를 `LINKED` + `todo_id` 설정
4. 기존에 연결돼 있었으나 본문에서 사라진 첨부는 **Soft Delete**
5. 다른 사용자의 첨부 ID가 섞여 있으면 **거부**(404)

### ④ 고아 파일 정리

`@Scheduled` 배치, 하루 1회. **상태별로 규칙이 다르다.**

| 대상 | 조건 | 처리 |
|---|---|---|
| `PENDING` | 생성 후 1시간 경과 | 파일이 없으므로 레코드만 정리 |
| `UPLOADED` | 생성 후 24시간 경과 | 실제 파일 삭제 + 레코드 Soft Delete |
| Soft Delete됨 | `deleted_at` 후 7일 경과 | 실제 파일 삭제 (유예 기간) |

> 본문에서 이미지를 지우고 저장하면 즉시 파일까지 지우는 것이 원문 설계인데, 그러면 되돌릴 수 없다.
> **레코드는 Soft Delete하고 파일 삭제는 유예한다.**
> `CLAUDE.md` 절대 규칙 5(물리 삭제 금지)는 DB 행에 대한 것이며, 파일 정리는 배치에서 별도로 다룬다.

`config/SchedulingConfig.java`에 `@EnableScheduling`을 새로 추가해야 한다. 현재 저장소에 없다.
EC2 단일 인스턴스를 전제로 한다 (다중 인스턴스면 중복 실행된다).

### ⑨ 신규 통합 테스트 9~12번

기존 방식을 그대로 따른다.
`@SpringBootTest + @AutoConfigureMockMvc + @ActiveProfiles("test") + @Transactional`,
`JwtTokenProvider.createAccessToken(userId)`로 **실제 JWT 발급** 후 `Authorization` 헤더. `@WithMockUser` 미사용.

| 번호 | 검증 |
|---|---|
| 9 | presign → upload → complete 전체 흐름과 `PENDING` → `UPLOADED` 상태 전이 |
| 10 | 타인 첨부 접근 시 **404**, 서명 토큰 위조·만료 거부 |
| 11 | 크기 초과 / `image/svg+xml` / 화이트리스트 밖 타입 거부 |
| 12 | 할일 저장 시 `LINKED` 승격, 본문에서 사라진 첨부 Soft Delete, **sanitize 이후 수집** 순서 |

---

## 11. 프론트엔드

### 파일 배치 — 기존 평면 구조 준수

```
src/types/attachment.ts             서버 DTO 대응 타입. 컴포넌트 내 인라인 재선언 금지
src/lib/attachments.ts              API 함수
src/hooks/useAttachments.ts         React Query 뮤테이션
src/lib/queryKeys.ts                attachmentKeys 추가
src/components/todo/TodoEditor.tsx  이미지 확장 + 툴바 버튼
src/lib/sanitize.ts                 img 허용
src/app/globals.css                 .tiptap-content img 스타일
```

> 원문의 `lib/api/attachments.ts`는 **존재하지 않는 디렉터리**다. `src/lib/`은 평면 구조다.

### ① DOMPurify 허용 목록 갱신 — 원문 누락

`src/lib/sanitize.ts`:

```ts
const ALLOWED_TAGS = ["p", "strong", "em", "ul", "ol", "li", "br", "a", "img"] as const;
const ALLOWED_ATTR = ["href", "rel", "target", "alt", "data-attachment-id"] as const;
```

**`src`는 넣지 않는다.** 서버 Safelist와 정확히 같은 집합을 유지한다.
`ALLOWED_URI_REGEXP`는 그대로 둔다 — `src`를 허용하지 않으므로 영향이 없다.

`globals.css`에 `.tiptap-content img` 스타일(최대 폭, 둥근 모서리, 업로드 중 자리 표시)을 추가한다.

### Tiptap 이미지 노드

`@tiptap/extension-image`(승인됨)를 `extend`해 `attachmentId` 속성을 추가한다.

```ts
Image.extend({
  addAttributes() {
    return {
      ...this.parent?.(),
      attachmentId: {
        default: null,
        parseHTML: (el) => el.getAttribute("data-attachment-id"),
        renderHTML: (attrs) =>
          attrs.attachmentId ? { "data-attachment-id": attrs.attachmentId } : {},
      },
    };
  },
})
```

`...this.parent?.()`를 빠뜨리면 `src`·`alt` 등 원래 속성이 통째로 사라진다.

툴바 이미지 버튼, 붙여넣기(paste), 드래그앤드롭 **세 경로 모두** 지원한다.

### ③ `uploadFile`만 `apiClient`를 우회한다 — 원문 누락

`src/lib/apiClient.ts`는 **모든** 요청에 `Authorization` 헤더와 `credentials: "include"`를 붙이고
401이면 refresh를 시도한다. 이걸 S3에 그대로 쏘면 서명이 깨지고 refresh 로직까지 엉킨다.

| 함수 | 사용 |
|---|---|
| `presignUpload` / `completeUpload` / `getViewUrls` / `deleteAttachment` | `apiClient` |
| **`uploadFile`** | **순수 `fetch` (또는 `XMLHttpRequest`)** |

`uploadFile`은 서버가 준 `uploadUrl`로 PUT을 보낼 뿐,
**로컬인지 S3인지 판단하는 로직을 두지 않는다.** 이 구분을 코드 주석으로 못박는다.

### 업로드 UX

1. 파일 선택 즉시 로컬 blob URL로 **임시 미리보기** 노드 삽입
2. presign → PUT → complete 순차 호출
3. 성공 시 노드의 `attachmentId`를 채우고 `src`를 실제 URL로 교체
4. 실패 시 노드 제거 + `sonner` 토스트로 안내

**`URL.createObjectURL`은 교체·실패·언마운트 시 반드시 `revokeObjectURL`로 해제한다.** 메모리 누수 지점이다.

### ⑪ 진행률 표시 — 원문의 구현 불가 지점

원문 7장 2번은 "진행률 표시"를 요구하지만 **`fetch`는 업로드 진행률을 제공하지 않는다.**
둘 중 하나를 고른다.

- **(권장) 진행률을 포기하고 불확정 스피너로 대체한다.** 5MB 상한이라 체감 시간이 짧다
- 진행률이 꼭 필요하면 `uploadFile`만 `XMLHttpRequest`로 구현한다 (`xhr.upload.onprogress`)

### 조회

할일 상세를 불러올 때 본문의 `data-attachment-id`를 모아
`POST /api/attachments/view-urls`로 **한 번에** 요청하고 `src`에 주입한다.

### 클라이언트 검증

업로드 전 파일 타입·크기를 미리 확인해 불필요한 요청을 막는다. **서버 검증은 그대로 유지한다.**

### `TodoForm`의 dirty 판정

`baselineHtml` 비교식이다. 업로드 완료로 노드 속성이 바뀌면 dirty가 되는 것이
**의도된 동작**이므로 별도 처리는 필요 없다.

---

## 12. 로컬 테스트 시나리오

구현 후 직접 확인하고 결과를 보고한다. **원문 10개 → 16개.**

| # | 확인 |
|---|---|
| 1 | 서버 기동 시 `todo-project/upload` 디렉터리가 자동 생성되는가 |
| 2 | 이미지 첨부 시 `upload/todos/{userId}/...` 경로에 파일이 실제로 생기는가 |
| 3 | `attachments` 테이블에 `status=PENDING`으로 행이 생기는가 |
| 4 | `complete` 후 `status=UPLOADED`가 되는가 |
| 5 | 할일 저장 후 `status=LINKED`, `todo_id`가 채워지는가 |
| 6 | 저장된 할일을 다시 열었을 때 이미지가 정상 표시되는가 |
| 7 | 본문에서 이미지를 지우고 저장하면 해당 첨부가 Soft Delete 되는가 |
| 8 | 5MB 초과 파일, `.exe` 파일 업로드가 거부되는가 |
| 9 | 다른 사용자의 `attachmentId`로 조회 시 **404**가 나는가 (403 아님) |
| 10 | `upload/` 폴더가 `todo-project` git status에 잡히지 않는가 |
| 11 | 저장 후 DB `content`에 `<img data-attachment-id>`가 **살아 있는가** (Jsoup 통과) |
| 12 | 화면 렌더 시 이미지가 **DOMPurify에 지워지지 않는가** |
| 13 | `image/svg+xml` 업로드가 거부되는가 |
| 14 | 만료·위조 서명 토큰으로 `/raw` 접근이 차단되는가 |
| 15 | 확장자는 `.png`인데 실제 내용이 다른 파일이 어떻게 처리되는가 |
| 16 | 저장된 HTML에 `src`가 **없는가** (URL 박제 방지) |

> 원문 9번의 "`storageKey`에 `../`를 넣은 요청"은 **응답에서 `storageKey`를 제거했으므로
> 클라이언트가 보낼 통로 자체가 없다.** 서버 내부 정규화 검증은 단위 테스트로 대체한다.

---

## 13. S3 전환

- `S3StorageService` 구현 + AWS SDK v2 의존성 추가 (BOM으로 버전 관리, 승인됨)
- `application-prod.properties` 프로파일로 기동해 12장 시나리오 전체 재확인
- **프론트 코드는 변경이 없어야 한다.** 수정이 필요하다면 추상화가 잘못된 것이므로 보고할 것

### presigned PUT의 함정 (원문 누락)

**`Content-Type`이 서명에 포함된다.** 프론트가 presign 요청 시 보낸 값과
**정확히 같은 값**을 PUT 헤더에 실어야 하며, 한 글자라도 다르면 403이다.
로컬 테스트에서는 드러나지 않고 S3 전환 시점에만 터진다.

### AWS 콘솔 설정 (별도 안내 필요)

- 버킷 **CORS** — `PUT`, `GET`, `HEAD` 허용 / `AllowedOrigin`에 로컬·Amplify 도메인 /
  **`AllowedHeader`에 `Content-Type` 필수**
- 퍼블릭 액세스 차단은 **켠 상태 유지**
- IAM은 해당 버킷의 `s3:PutObject`, `s3:GetObject`, `s3:DeleteObject`만 (최소 권한)

---

## 14. 진행 순서

각 단계가 끝나면 **멈추고 확인받는다.** DoD는 실제 명령의 종료 코드로 판정한다.

| 단계 | 내용 | DoD |
|---|---|---|
| 0 | 정본 문서 개정 (PRD / CLAUDE.md / ROADMAP / SCHEMA / CHECKLIST) | 문서 상호 참조 일치 |
| 1 | `.yml` → `.properties` 전환 + 연쇄 수정 | `./mvnw spotless:apply test` |
| 2 | `db/add-attachments.sql` + Attachment 엔티티 / 리포지토리 | `./mvnw test` |
| 3 | `StorageService` + `LocalStorageService` + 서명 토큰 + AttachmentService / Controller | `./mvnw spotless:apply test` |
| 4 | TodoService 연결 로직 + 정리 배치 + **Jsoup Safelist에 img** | `./mvnw test` (테스트 9~12번 포함) |
| 5 | 프론트 타입 / API / 훅 + Tiptap 이미지 노드 + **DOMPurify 갱신** | `npm run check && npm run build` |
| 6 | 로컬 통합 테스트 (12장 16개 전체) | 수동 확인 후 결과 보고 |
| 7 | `S3StorageService` + AWS SDK + 전환 검증 | 프론트 무변경 확인 |

---

## 15. 규칙

- 코드 주석은 **한글**로 작성한다. 코드 식별자는 영문이다
- **`.properties` 값에는 ASCII만 쓴다.** 한글은 주석에만 (ISO-8859-1 로딩)
- 소유권 실패는 **404**다. 403을 쓰지 않는다
- 삭제는 Soft Delete만. 물리 삭제 금지. 실제 파일 삭제는 정리 배치에서 유예 후 수행한다
- 자격증명은 어떤 형태로도 소스에 남기지 않는다
- 설정 파일은 **`.properties` 형식만** 사용한다
- 백엔드 커밋 전 `./mvnw spotless:apply`
- **폴리레포 3개는 각각 커밋한다.** 하나의 커밋으로 묶이지 않는다
- Conventional Commits. 프론트만 husky + commitlint로 자동 검증되므로 백엔드·문서는 직접 지킨다
- 기존 파일을 수정할 때 관련 없는 코드를 함께 리팩터링하지 않는다
- 라이브러리 API가 확실하지 않으면 **공식 문서를 확인한다.** Spring Boot 4 / Next.js 16 /
  Tailwind 4 / Tiptap 3는 이전 메이저와 문법이 다르다

### 승인된 신규 의존성

| 패키지 | 사유 | 도입 단계 |
|---|---|---|
| `@tiptap/extension-image` | Tiptap 3.30.6 StarterKit에 이미지 노드가 없다 | 5단계 |
| AWS SDK v2 `s3` (`S3Presigner`는 별도 모듈이 아니라 이 아티팩트에 포함) | S3 스토리지 구현 | 7단계 |

이 둘 외에는 임의로 추가하지 않는다. 필요하면 사유와 함께 제안하고 승인을 받는다.

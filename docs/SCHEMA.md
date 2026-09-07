# SCHEMA.md — DB 스키마 및 엔티티

**저장소**: `todo-backend`
**최종 수정**: Phase 2(도메인 & DB) 완료 시점

이 문서는 `CLAUDE.md`가 정의한 규칙(패키지명 `com.example`, DB명 `todolist_db`/`todolist_db_test`, Soft Delete, 엔티티 Setter 금지)을 실제 테이블·엔티티 설계로 옮긴 것이다. 아래 내용은 로컬 `todolist_db`에 실제로 생성된 스키마(`information_schema` 조회 결과)와 동기화되어 있다.

---

## 1. 개요

- DB: PostgreSQL. 로컬 `todolist_db`, 테스트 `todolist_db_test` (`CLAUDE.md` 절대 규칙 2)
- 스키마 생성: JPA/Hibernate `ddl-auto` (local: `update`, test: `create-drop`, prod: `validate` — Phase 11에서 수동 적용 후 고정)
- 시간대: 모든 타임스탬프는 UTC로 저장한다. JVM 기본 타임존을 `TimeZone.setDefault(UTC)`로 고정하고(`TodoBackendApplication` static 블록), `hibernate.jdbc.time_zone: UTC`를 병행한다 — 전자가 없으면 `LocalDateTime.now()`가 로컬 OS 시간대(KST)로 채워지는 채로 저장되는 문제가 있다(Phase 2에서 실측 확인).
- 패키지: `com.example.todoapp.domain` 아래 엔티티·enum·Repository를 모두 둔다 (계층형 구조. `CLAUDE.md` 4장도 계층형으로 확정됐다).

---

## 2. 공통 필드 (BaseEntity)

`@MappedSuperclass`. `User`, `Todo`가 상속한다.

| 컬럼 | 타입 | NULL | 설명 |
|---|---|---|---|
| `created_at` | `timestamp` | NOT NULL | `@CreatedDate`로 자동 기록, 수정 불가(`updatable=false`) |
| `updated_at` | `timestamp` | NOT NULL | `@LastModifiedDate`로 자동 갱신 |
| `deleted_at` | `timestamp` | NULL | Soft Delete 마커. `NULL`이면 삭제되지 않은 상태. `softDelete()` 메서드로만 채워진다 |

물리 삭제(`DELETE FROM`)는 어떤 테이블에도 사용하지 않는다 (`CLAUDE.md` 절대 규칙 5).

---

## 3. `users` 테이블

엔티티: `com.example.todoapp.domain.User`

| 컬럼 | 타입 | NULL | 제약 | 설명 |
|---|---|---|---|---|
| `id` | `bigint` | NOT NULL | PK, IDENTITY | |
| `email` | `varchar(254)` | NOT NULL | UNIQUE | 로그인 ID |
| `password` | `varchar(60)` | NULL | | BCrypt 해시. 구글 전용 계정은 `NULL` (PRD F-07) |
| `nickname` | `varchar(50)` | NOT NULL | | 1~50자 |
| `provider` | `varchar(20)` | NOT NULL | CHECK (`LOCAL`,`GOOGLE`) | `AuthProvider` enum, `EnumType.STRING` |
| `created_at` / `updated_at` / `deleted_at` | | | | BaseEntity 공통 필드 |

생성 경로는 `User.createLocal(email, encodedPassword, nickname)` / `User.createGoogle(email, nickname)` 정적 팩토리로만 제한된다(Setter 없음).

---

## 4. `todos` 테이블

엔티티: `com.example.todoapp.domain.Todo`

| 컬럼 | 타입 | NULL | 제약 | 설명 |
|---|---|---|---|---|
| `id` | `bigint` | NOT NULL | PK, IDENTITY | |
| `user_id` | `bigint` | NOT NULL | FK → `users.id` | `@ManyToOne(fetch = LAZY)` |
| `title` | `varchar(200)` | NOT NULL | | 1~200자 (PRD F-11) |
| `content` | `text` | NULL | | Tiptap HTML. 저장 전 Jsoup sanitize는 Phase 4에서 적용 |
| `priority` | `varchar(20)` | NOT NULL | CHECK (`LOW`,`MEDIUM`,`HIGH`) | `Priority` enum, 기본값 `MEDIUM` |
| `due_date` | `date` | NULL | | |
| `completed` | `boolean` | NOT NULL | 기본 `false` | |
| `completed_at` | `timestamp` | NULL | | 완료 시 기록, 해제 시 `NULL`로 되돌림 (PRD F-20) |
| `created_at` / `updated_at` / `deleted_at` | | | | BaseEntity 공통 필드 |

**인덱스**:
- `idx_todos_user_deleted` — `(user_id, deleted_at)` 복합 인덱스. `TodoRepository`의 모든 조회가 "특정 `user_id` + `deleted_at IS NULL`" 패턴이므로, `user_id`를 선행 컬럼으로 둬 B-tree 접두사 매칭을 활용한다. `@Table(indexes=...)`로 자동 생성됨.
- `idx_todos_title_lower` — `LOWER(title)` 함수 기반 인덱스. 제목 대소문자 무시 검색(`TodoRepository.search`의 `lower(t.title) like lower(concat('%', :keyword, '%'))`)을 지원한다. JPA 애노테이션으로 함수 표현식 인덱스를 만들 수 없어 `db/add-title-index.sql`로 수동 관리한다(Phase 4, 이 프로젝트는 Flyway/Liquibase 미사용). 로컬 `todolist_db`에 적용 완료, 신규 환경 구성 시 이 스크립트를 실행해야 한다.

**소유권 검증**: 모든 조회는 `user_id`를 쿼리 조건에 포함해야 한다(PRD NF-02, `CLAUDE.md` 절대 규칙 4). 애플리케이션 코드로 사후 필터링하지 않는다.

상태 변경은 다음 메서드로만 가능하다(Setter 없음):
- `complete()` / `uncomplete()` — `completed`·`completed_at`만 변경
- `updateContent(title, content, priority, dueDate)` — 완료 상태는 건드리지 않음 (ROADMAP Phase 4 TODO-10: PUT이 완료 상태를 덮어쓰지 않음)
- `softDelete()` — `deleted_at` 기록 (BaseEntity 공통)

---

## 5. `attachments` 테이블

엔티티: `com.example.todoapp.domain.Attachment`

Tiptap 본문에 첨부한 이미지의 메타데이터. 파일 실체는 스토리지(로컬 디렉터리 또는 S3)에 있고
이 테이블은 그 참조와 상태만 관리한다 (PRD F-46 ~ F-51).

| 컬럼 | 타입 | NULL | 제약 | 설명 |
|---|---|---|---|---|
| `id` | `bigint` | NOT NULL | PK, IDENTITY | |
| `todo_id` | `bigint` | NULL | FK → `todos.id` | `@ManyToOne(fetch = LAZY)`. **NULL 허용** — 아래 참고 |
| `user_id` | `bigint` | NOT NULL | FK → `users.id` | `@ManyToOne(fetch = LAZY)`. 업로더. 소유권 검증의 기준 |
| `storage_type` | `varchar(20)` | NOT NULL | CHECK (`LOCAL`,`S3`) | `StorageType` enum |
| `storage_key` | `varchar(512)` | NOT NULL | UNIQUE | 로컬 상대경로 또는 S3 객체 키. **서버만 생성**하고 API 응답에 노출하지 않는다 (PRD NF-33) |
| `original_filename` | `varchar(255)` | NOT NULL | | 원본 파일명 |
| `content_type` | `varchar(100)` | NOT NULL | | 화이트리스트 검증을 통과한 값 (PRD F-47) |
| `file_size` | `bigint` | NOT NULL | | bytes. presign 시 신고값으로 채우고 complete 시 실측값으로 덮어쓴다 |
| `status` | `varchar(20)` | NOT NULL | CHECK (`PENDING`,`UPLOADED`,`LINKED`) | `AttachmentStatus` enum. 아래 참고 |
| `created_at` / `updated_at` / `deleted_at` | | | | BaseEntity 공통 필드 |

**인덱스**:
- `idx_attachments_todo_deleted` — `(todo_id, deleted_at)`. 할 일 상세에서 첨부를 모을 때 쓴다.
- `idx_attachments_user_deleted` — `(user_id, deleted_at)`. 소유권 검증 조회용.
- `idx_attachments_status_created` — `(status, created_at)`. 고아 파일 정리 배치용.

**`todo_id`가 NULL 허용인 이유**: 에디터에서는 할 일을 저장하기 *전에* 이미지가 먼저 업로드된다.
업로드 시점에는 연결할 할 일이 아직 없다.

**상태 전이**:

| 상태 | 의미 | 전이 시점 |
|---|---|---|
| `PENDING` | 레코드만 있고 파일이 없다 | `POST /api/attachments/presign` 응답 시 |
| `UPLOADED` | 파일은 확정됐으나 아직 할 일에 연결되지 않았다 | `POST /api/attachments/{id}/complete` 성공 시 |
| `LINKED` | 할 일 본문에 실제로 존재한다 | 할 일 저장/수정 시 |

`PENDING`과 `UPLOADED`를 구분하는 이유는 정리 배치가 두 경우에 서로 다른 정책을 적용하기 때문이다.
`PENDING`은 파일이 없으므로 레코드만 정리하면 되고, `UPLOADED`는 실제 파일까지 지워야 한다.

**저장 키 규칙**: `todos/{userId}/{yyyy}/{MM}/{uuid}.{ext}` (로컬·S3 동일)

**마이그레이션**: local은 `ddl-auto: update`가 엔티티에서 생성한다. prod는 `validate`이므로
`todo-backend/db/add-attachments.sql`을 수동 실행한다 (`db/add-title-index.sql`과 같은 관행).

---

## 6. 패키지 구조

```
com.example
├── TodoBackendApplication.java   # @EnableJpaAuditing, JVM 타임존 UTC 고정(static 블록)
├── domain/
│   ├── BaseEntity.java           # 공통 감사 필드 + softDelete()
│   ├── User.java
│   ├── Todo.java
│   ├── Priority.java             # enum: LOW, MEDIUM, HIGH
│   ├── AuthProvider.java         # enum: LOCAL, GOOGLE
│   ├── Attachment.java
│   ├── AttachmentStatus.java     # enum: PENDING, UPLOADED, LINKED
│   ├── StorageType.java          # enum: LOCAL, S3
│   ├── UserRepository.java
│   ├── TodoRepository.java
│   └── AttachmentRepository.java
├── service/                      # Phase 3~4에서 채워짐
├── controller/                   # Phase 3~4에서 채워짐
├── dto/                          # Phase 3~4에서 채워짐
├── config/
│   └── SecurityConfig.java       # Phase 1 임시본, Phase 3에서 전체 교체 예정
└── exception/                    # Phase 3~4에서 채워짐
```

`domain` 패키지 안에 엔티티·enum·Repository를 함께 둔다. 이 계층형 구조가 확정본이며 `CLAUDE.md` 4장도 같은 내용으로 갱신됐다.

---

## 7. Repository 조회 메서드

| Repository | 메서드 | 용도 |
|---|---|---|
| `UserRepository` | `findByEmailAndDeletedAtIsNull(email)` | 로그인, 이메일 중복 체크 |
| `UserRepository` | `existsByEmailAndDeletedAtIsNull(email)` | 회원가입 시 중복 검사 |
| `TodoRepository` | `findByIdAndUser_IdAndDeletedAtIsNull(id, userId)` | 단건 조회 (소유권 강제) |
| `TodoRepository` | `findAllByUser_IdAndDeletedAtIsNull(userId, pageable)` | 목록 조회 (소유권 강제) |
| `AttachmentRepository` | `findByIdAndUser_IdAndDeletedAtIsNull(id, userId)` | 단건 조회 (소유권 강제) |
| `AttachmentRepository` | `findAllByIdInAndUser_IdAndDeletedAtIsNull(ids, userId)` | 본문 첨부 일괄 조회 (N+1 회피) |
| `AttachmentRepository` | `findAllByTodo_IdAndDeletedAtIsNull(todoId)` | 할 일에 연결된 첨부 |
| `AttachmentRepository` | `findAllByStatusAndCreatedAtBefore(status, cutoff)` | 고아 파일 정리 배치 |

페이지네이션 필터(`completed`)·검색(`keyword`)·정렬 화이트리스트가 포함된 커스텀 쿼리는 Phase 4(Todo API)에서 추가한다.

# SCHEMA.md — DB 스키마 및 엔티티

**저장소**: `todo-backend`
**최종 수정**: Phase 2(도메인 & DB) 완료 시점

이 문서는 `CLAUDE.md`가 정의한 규칙(패키지명 `com.example`, DB명 `todolist_db`/`todolist_db_test`, Soft Delete, 엔티티 Setter 금지)을 실제 테이블·엔티티 설계로 옮긴 것이다. 아래 내용은 로컬 `todolist_db`에 실제로 생성된 스키마(`information_schema` 조회 결과)와 동기화되어 있다.

---

## 1. 개요

- DB: PostgreSQL. 로컬 `todolist_db`, 테스트 `todolist_db_test` (`CLAUDE.md` 절대 규칙 2)
- 스키마 생성: JPA/Hibernate `ddl-auto` (local: `update`, test: `create-drop`, prod: `validate` — Phase 11에서 수동 적용 후 고정)
- 시간대: 모든 타임스탬프는 UTC로 저장한다. JVM 기본 타임존을 `TimeZone.setDefault(UTC)`로 고정하고(`TodoBackendApplication` static 블록), `hibernate.jdbc.time_zone: UTC`를 병행한다 — 전자가 없으면 `LocalDateTime.now()`가 로컬 OS 시간대(KST)로 채워지는 채로 저장되는 문제가 있다(Phase 2에서 실측 확인).
- 패키지: `com.example.domain` 아래 엔티티·enum·Repository를 모두 둔다 (Phase 1~2는 계층형 골격 유지, `CLAUDE.md` 4장의 기능별 재편은 이후 Phase 검토 대상).

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

엔티티: `com.example.domain.User`

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

엔티티: `com.example.domain.Todo`

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

**인덱스**: `idx_todos_user_deleted` — `(user_id, deleted_at)` 복합 인덱스. `TodoRepository`의 모든 조회가 "특정 `user_id` + `deleted_at IS NULL`" 패턴이므로, `user_id`를 선행 컬럼으로 둬 B-tree 접두사 매칭을 활용한다.

**소유권 검증**: 모든 조회는 `user_id`를 쿼리 조건에 포함해야 한다(PRD NF-02, `CLAUDE.md` 절대 규칙 4). 애플리케이션 코드로 사후 필터링하지 않는다.

상태 변경은 다음 메서드로만 가능하다(Setter 없음):
- `complete()` / `uncomplete()` — `completed`·`completed_at`만 변경
- `updateContent(title, content, priority, dueDate)` — 완료 상태는 건드리지 않음 (ROADMAP Phase 4 TODO-10: PUT이 완료 상태를 덮어쓰지 않음)
- `softDelete()` — `deleted_at` 기록 (BaseEntity 공통)

---

## 5. 패키지 구조

```
com.example
├── TodoBackendApplication.java   # @EnableJpaAuditing, JVM 타임존 UTC 고정(static 블록)
├── domain/
│   ├── BaseEntity.java           # 공통 감사 필드 + softDelete()
│   ├── User.java
│   ├── Todo.java
│   ├── Priority.java             # enum: LOW, MEDIUM, HIGH
│   ├── AuthProvider.java         # enum: LOCAL, GOOGLE
│   ├── UserRepository.java
│   └── TodoRepository.java
├── service/                      # Phase 3~4에서 채워짐
├── controller/                   # Phase 3~4에서 채워짐
├── dto/                          # Phase 3~4에서 채워짐
├── config/
│   └── SecurityConfig.java       # Phase 1 임시본, Phase 3에서 전체 교체 예정
└── exception/                    # Phase 3~4에서 채워짐
```

`domain` 패키지 안에 엔티티·enum·Repository를 함께 둔다. `CLAUDE.md` 4장의 기능별(package-by-feature) 재편은 Phase 1 스캐폴딩이 계층형으로 이미 굳어진 상태라 이번 Phase에서는 다루지 않았다.

---

## 6. Repository 조회 메서드

| Repository | 메서드 | 용도 |
|---|---|---|
| `UserRepository` | `findByEmailAndDeletedAtIsNull(email)` | 로그인, 이메일 중복 체크 |
| `UserRepository` | `existsByEmailAndDeletedAtIsNull(email)` | 회원가입 시 중복 검사 |
| `TodoRepository` | `findByIdAndUser_IdAndDeletedAtIsNull(id, userId)` | 단건 조회 (소유권 강제) |
| `TodoRepository` | `findAllByUser_IdAndDeletedAtIsNull(userId, pageable)` | 목록 조회 (소유권 강제) |

페이지네이션 필터(`completed`)·검색(`keyword`)·정렬 화이트리스트가 포함된 커스텀 쿼리는 Phase 4(Todo API)에서 추가한다.

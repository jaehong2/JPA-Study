# JPA 스터디 메모

## 목차

### 엔티티 매핑
- 객체와 테이블 매핑
- DB 스키마 자동 생성
- 필드와 컬럼 매핑
- 기본 키 매핑

---

## 목표

### 객체와 테이블 설계 매핑
- 객체와 테이블을 제대로 설계하고 매핑
- 기본 키와 외래키 매핑
- 연관관계 매핑
  - 1:N (일대다)
  - N:1 (다대일)
  - 1:1 (일대일)
  - N:M (다대다)

### JPA 내부 동작 방식 이해
- JPA의 내부 동작 이해
- JPA가 어떤 SQL을 만들어서 언제 실행하는지 이해

---

## 메모

### 객체를 관계형 DB에 관리하는 시대

- 현실: 객체 지향 언어(Java) + 관계형 DB(MySQL 등) 조합이 대세
- 문제: **SQL 중심 개발** → 개발자가 SQL 매퍼 역할을 반복 수행
- 핵심 문제: **객체 vs 관계형 DB 패러다임 불일치**
  - 객체는 참조(`member.getTeam()`)로 탐색
  - DB는 외래키 + JOIN으로 탐색
  - → 개발자가 중간에서 수동으로 변환해야 함
- **SQL 의존적 개발** 불가피
  - 필드 하나 추가 시 → 관련 SQL 전부 수정
  - 진정한 의미의 객체 지향 설계 불가능

### 객체 vs 관계형 DB의 차이

| 차이점 | 객체 | 관계형 DB |
|---|---|---|
| **상속** | O (자연스럽게 지원) | X (슈퍼타입/서브타입으로 흉내) |
| **연관관계** | 참조(`getTeam()`) | 외래키 + JOIN |
| **데이터 타입** | 다양한 객체 타입 | 제한된 컬럼 타입 |
| **데이터 식별** | 동일성(`==`) / 동등성(`.equals()`) | 기본키(PK) |

### 객체 그래프 탐색

- 객체는 참조를 통해 자유롭게 탐색 가능
  ```java
  member.getTeam().getOrder().getItem() // 자유롭게 탐색
  ```
- DB는 처음 실행한 SQL에 따라 탐색 범위가 결정됨
  - JOIN 안 한 테이블은 조회 불가 → `NullPointerException` 위험
- 결론: **SQL 결과에 따라 객체 그래프 탐색 범위가 제한됨** (신뢰할 수 없는 엔티티)

### 계층형 아키텍처

- 진정한 의미의 계층 분할이 어렵다
  - SQL에 의존하는 순간 → DAO가 어떤 쿼리를 날리는지 직접 열어봐야 신뢰 가능
  - 물리적 계층 분리는 되지만, **논리적으로는 SQL에 종속**
- **객체답게 모델링할수록 매핑 작업만 늘어남**
  - 객체 지향적으로 설계 → SQL 변환 코드 증가
  - 결국 객체 지향을 포기하고 SQL에 맞춰 설계하게 됨

---

### JPA / ORM

- **ORM** (Object-Relational Mapping)
  - 객체와 관계형 DB를 자동으로 매핑해주는 기술
  - 개발자는 SQL 대신 객체를 다룸 → ORM이 SQL 생성/실행

- **JPA** (Java Persistence API)
  - Java 진영의 ORM 표준 인터페이스 (명세)
  - 구현체: **Hibernate** (가장 많이 사용), EclipseLink 등

- **핵심**: JPA가 패러다임 불일치 문제를 해결해줌
- **동작 위치**: `App → JPA → JDBC → DB`
  - 개발자가 JPA에 명령 → JPA가 JDBC API로 SQL 생성 → DB 전달

### JPA를 왜 사용해야 하는가?

- **생산성** - SQL 작성 불필요, CRUD 자동 처리
- **유지보수** - 필드 변경 시 SQL 수정 불필요
- **패러다임 불일치 해결**
  - **상속**: `persist(album)` 한 번으로 INSERT 2개(Item, Album) 자동 처리
  - **연관관계**: 외래키 대신 참조로 저장/조회 (`member.setTeam(team)`)
  - **객체 그래프 탐색**: 신뢰 가능한 엔티티 보장, 지연 로딩으로 자유롭게 탐색
  - **동일성 보장**: 같은 트랜잭션 내 동일 엔티티 조회 시 `==` 비교 true (1차 캐시 덕분)
  - **쓰기 지연**: 트랜잭션 커밋 전까지 INSERT SQL을 모아뒀다가 JDBC Batch로 한 번에 전송
  - **지연 로딩**: 연관 객체를 실제 사용하는 시점에 SQL 실행 (필요할 때만 조회)
  - **즉시 로딩**: 연관 객체를 함께 JOIN으로 한 번에 조회
  - > 실무 팁: 기본은 **지연 로딩**으로 세팅 → 성능 이슈 확인 후 즉시 로딩으로 변경
- **성능 최적화**
  - 1차 캐시 (같은 트랜잭션 내 동일 쿼리 재사용)
  - 지연 로딩 (필요할 때만 SQL 실행)
- **DB 방언(Dialect)** - DB 변경 시 코드 수정 없이 설정만 변경

---

### JPA 설정 - persistence.xml

- 경로: `src/main/resources/META-INF/persistence.xml` (JPA가 자동 인식)
- `persistence-unit name` → EntityManagerFactory 생성 시 사용할 이름

**필수 속성**
| 속성 | 설명 |
|---|---|
| `javax.persistence.jdbc.driver` | JDBC 드라이버 |
| `javax.persistence.jdbc.url` | DB 접속 URL |
| `javax.persistence.jdbc.user/password` | DB 계정 |
| `hibernate.dialect` | DB 방언 설정 (H2, MySQL, Oracle 등) |

**옵션 속성**
| 속성 | 설명 |
|---|---|
| `hibernate.show_sql` | 실행 SQL 출력 |
| `hibernate.format_sql` | SQL 포맷팅 출력 |
| `hibernate.use_sql_comments` | SQL 주석 출력 |
| `hibernate.jdbc.batch_size` | 쓰기 지연 batch 크기 |
| `hibernate.hbm2ddl.auto` | DDL 자동 생성 (create/update/validate 등) |

### 데이터베이스 방언 (Dialect)

- JPA는 특정 DB에 종속되지 않음
- DB마다 SQL 문법이 조금씩 다름 (방언)
  - 페이징: MySQL → `LIMIT`, Oracle → `ROWNUM`
  - 문자열 자르기: MySQL → `SUBSTRING()`, Oracle → `SUBSTR()`
- `hibernate.dialect` 설정으로 DB 변경 시 코드 수정 없이 대응 가능
  - H2 → `H2Dialect`
  - MySQL → `MySQL8Dialect`
  - Oracle → `OracleDialect`

### JPA 구동 방식

```
persistence.xml → Persistence → EntityManagerFactory → EntityManager
```

1. `Persistence.createEntityManagerFactory("hello")` → persistence.xml 설정 읽어서 EMF 생성
2. `emf.createEntityManager()` → 요청마다 EntityManager 생성
3. `em.getTransaction()` → 트랜잭션 획득 후 `begin()`
4. 로직 수행 → `commit()` / 예외 시 `rollback()`
5. `em.close()` → EntityManager 반드시 닫기
6. `emf.close()` → 앱 종료 시 Factory 닫기

> **주의**
> - `EntityManagerFactory` → 앱 전체에서 **딱 1개** 생성
> - `EntityManager` → 요청(트랜잭션)마다 **새로 생성** 후 사용, **쓰레드 간 공유 절대 X**
> - JPA의 모든 데이터 변경은 **트랜잭션 안에서** 실행

### 변경 감지 (Dirty Checking)

- 조회한 엔티티의 값만 바꾸면 **persist() 호출 없이 자동 UPDATE**
- 동작 원리: 최초 조회 시 **스냅샷 저장** → 커밋 시점에 비교 → 변경 시 UPDATE SQL 자동 실행
```java
Member findMember = em.find(Member.class, 1L);
findMember.setName("HelloJPA"); // 이것만으로 충분
tx.commit(); // 자동 UPDATE 실행
```
> 마치 Java 컬렉션처럼 값만 바꾸면 자동 반영

**변경 감지 동작 순서**
1. `flush()` 호출
2. 엔티티와 스냅샷 비교
3. 변경된 엔티티 → UPDATE SQL을 쓰기 지연 저장소에 추가
4. 쓰기 지연 저장소의 SQL flush (DB 전송)
5. `commit()`

**플러시 하는 방법**
- `em.flush()` - 직접 호출
- 트랜잭션 커밋 - 자동 호출
- JPQL 쿼리 실행 - 자동 호출 (쿼리 실행 전 DB 동기화 보장)

**flush 핵심 이유**
- 커밋 전에 **영속성 컨텍스트와 DB의 데이터 불일치 방지**
- flush ≠ commit → flush 후 롤백 시 전부 취소됨
- flush = SQL 전송 / commit = 트랜잭션 확정 (별개의 개념)
- **트랜잭션 단위가 중요** → 커밋 직전에만 동기화하면 됨

---

## 엔티티 매핑

### 엔티티 매핑 소개

| 매핑 | 어노테이션 |
|---|---|
| 객체와 테이블 매핑 | `@Entity`, `@Table` |
| 필드와 컬럼 매핑 | `@Column` |
| 기본 키 매핑 | `@Id` |
| 연관관계 매핑 | `@ManyToOne`, `@JoinColumn` |

### @Entity 주의사항

- **기본 생성자 필수** (public 또는 protected)
- `final` 클래스, `enum`, `interface`, `inner` 클래스 사용 X
- 저장할 필드에 `final` 사용 X

### 스키마 자동 생성 (DDL)

- `hibernate.hbm2ddl.auto` 옵션으로 설정

| 옵션 | 설명 |
|---|---|
| `create` | 기존 테이블 삭제 후 재생성 |
| `create-drop` | 종료 시 테이블 DROP |
| `update` | 변경분만 반영 |
| `validate` | 엔티티-테이블 매핑 검증만 |
| `none` | 사용 안 함 |

> **운영 환경에서 절대 사용 X** → 개발/로컬/테스트 환경에서만 사용

### 필드와 컬럼 매핑 어노테이션

| 어노테이션 | 설명 |
|---|---|
| `@Column` | 컬럼 매핑 (name, nullable, length 등) |
| `@Enumerated` | enum 타입 매핑 → **반드시 `EnumType.STRING` 사용** |
| `@Temporal` | 날짜 타입 매핑 (DATE/TIME/TIMESTAMP) → Java8 이상은 `LocalDate`, `LocalDateTime` 사용 권장 |
| `@Lob` | 대용량 데이터 매핑 → 필드 타입이 문자(String)면 **CLOB**, 나머지는 **BLOB** |
| `@Transient` | 특정 필드를 컬럼에 매핑하지 않음 (DB 저장 X, 메모리에서만 사용) |

### @Column 속성

| 속성 | 설명 | 기본값 |
|---|---|---|
| `name` | 컬럼명 지정 | 필드명 |
| `nullable` | null 허용 여부 | true |
| `unique` | 유니크 제약 조건 | false |
| `length` | 문자 길이 (String만) | 255 |
| `insertable` | INSERT 포함 여부 | true |
| `updatable` | UPDATE 포함 여부 | true |
| `columnDefinition` | 컬럼 DDL 직접 지정 | - |
| `precision / scale` | BigDecimal 소수점 자리 | - |

> **주의**: `unique` 속성은 제약조건 이름이 랜덤 생성되어 운영에서 알아보기 어려움 → `@Table(uniqueConstraints=...)` 사용 권장

### 기본 키 매핑

**매핑 방법**
- **직접 할당**: `@Id` 만 사용
- **자동 생성**: `@Id` + `@GeneratedValue`

| 전략 | 설명 | 주로 사용 DB |
|---|---|---|
| `IDENTITY` | DB에 위임 (AUTO_INCREMENT) → `em.persist()` 시점에 즉시 INSERT 실행 **(ID를 DB가 생성하므로 INSERT 전에 ID를 알 수 없음 → 쓰기 지연 불가)** | MySQL, PostgreSQL |
| `SEQUENCE` | DB 시퀀스 사용 (`@SequenceGenerator`) | Oracle, PostgreSQL |
| `TABLE` | 키 생성 전용 테이블 사용 (모든 DB 가능, 성능 이슈) | - |
| `AUTO` | DB 방언에 따라 자동 선택 (기본값) | - |

**@SequenceGenerator 속성**
```java
@SequenceGenerator(
    name = "MEMBER_SEQ_GENERATOR",
    sequenceName = "MEMBER_SEQ", // DB 시퀀스명
    initialValue = 1,            // 시작값
    allocationSize = 50          // 한 번에 가져올 크기 (성능 최적화)
)
```
> `allocationSize` 기본값 50 → DB 시퀀스를 50개씩 미리 가져와 메모리에서 사용 (네트워크 비용 절감)

**SEQUENCE 전략 동작 순서**
1. `em.persist()` 호출
2. DB 시퀀스에서 다음 값 조회 (`SELECT NEXTVAL`)
3. 조회한 값을 ID로 세팅 후 영속성 컨텍스트에 저장
4. 커밋 시점에 INSERT SQL 실행 (쓰기 지연 동작 O)

### 권장 식별자 전략

**기본 키 제약 조건**
- null 아님
- 유일
- **변하면 안 됨**

> 미래까지 이 조건을 만족하는 **자연키(주민번호, 전화번호 등)는 사용 X**
> → 비즈니스 키는 언제든 바뀔 수 있음

**권장**: `Long형` + `대체키(UUID 등)` + `키 생성 전략(@GeneratedValue)` 사용

### 준영속 상태

- 영속 상태의 엔티티가 영속성 컨텍스트에서 **분리(detached)** 된 상태
- JPA가 관리하지 않음 → **변경 감지, 쓰기 지연 등 기능 동작 X**

**준영속 상태로 만드는 방법**
- `em.detach(entity)` - 특정 엔티티만 분리
- `em.clear()` - 영속성 컨텍스트 전체 초기화
- `em.close()` - 영속성 컨텍스트 종료

---

## 영속성 컨텍스트

### JPA에서 가장 중요한 2가지
1. **객체와 관계형 DB 매핑** (설계 영역)
2. **영속성 컨텍스트** (런타임 동작 영역)

### 영속성 컨텍스트란?
- **엔티티를 영구 저장하는 환경** (논리적 개념, 눈에 보이지 않음)
- `EntityManager`를 통해 영속성 컨텍스트에 접근
- `em.persist(entity)` → DB 저장이 아닌 **영속성 컨텍스트에 저장**

### 엔티티의 생명주기

| 상태 | 설명 |
|---|---|
| **비영속 (new)** | 영속성 컨텍스트와 무관, 순수 객체 상태 |
| **영속 (managed)** | 영속성 컨텍스트에 관리되는 상태 (`em.persist()`, `em.find()`) |
| **준영속 (detached)** | 영속성 컨텍스트에서 분리된 상태 (`em.detach()`) |
| **삭제 (removed)** | 삭제된 상태 (`em.remove()`) |

```
new → persist() → [영속] → detach() → [준영속]
                         → remove()  → [삭제]
```
> 영속 상태가 된다고 SQL이 바로 실행되는 게 아님 → **트랜잭션 커밋 시점**에 SQL 실행 (쓰기 지연)

### 엔티티 조회 - 1차 캐시

- 영속성 컨텍스트 내부에 **1차 캐시** 존재 (Map 구조: `@Id` → 엔티티)
- `em.find()` 동작 순서
  1. 1차 캐시에서 먼저 조회
  2. 있으면 → **캐시에서 바로 반환** (SQL 없음)
  3. 없으면 → **DB 조회** → 1차 캐시에 저장 후 반환
- 트랜잭션이 끝나면 1차 캐시도 초기화 (트랜잭션 범위 내에서만 유효)

### JPQL

- **테이블이 아닌 객체(엔티티)를 대상으로 쿼리**
- JPA가 JPQL → DB에 맞는 SQL로 변환해서 실행
- DB 방언만 바꾸면 다른 DB에서도 동작

```java
// SQL: SELECT * FROM Member WHERE username LIKE '%kim%'
em.createQuery("select m from Member m where m.username like '%kim%'")
  .getResultList();
```

| | JPQL | SQL |
|---|---|---|
| 대상 | 엔티티 객체 | DB 테이블 |
| 이식성 | DB 독립적 | DB 종속적 |

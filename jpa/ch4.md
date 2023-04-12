# 엔티티 매핑

## 1. @Entity
클래스에 @Entity가 붙어야 JPA가 관리한다.

### 1.1 @Entity 속성
- `@Entity.name`: JPA에서 사용할 엔티티 이름 지정 (생략시 클래스의 이름을 사용)

### 1.2 @Entity 적용시 주의사항
- 기본 생성자 필수
- final 클래스, enum, interface, inner 클래스에는 사용할 수 없다.
- 저장할 필드에 final을 사용하면 안된다.

## 2. @Table
엔티티와 매핑할 테이블을 지정한다. (생략시 매핑한 엔티티 이름을 테이블 이름으로 사용)

### 2.1 @Table 속성
- `@Table.name`: 매핑할 테이블 이름 지정 (생략시 엔티티 이름을 사용)
- `@Table.catalog`: catalog 기능이 있는 DB에서 catalog를 매핑
- `@Table.schema`: schema 기능이 있는 DB에서 schema를 매핑
- `@Table.uniqueConstraints`: DDL 생성 시에 여러 유니크 제약조건을 생성 (스키마 자동 생성 기능을 사용해서 DDL을 생성할 때만 사용 가능)

## 3. DB 스키마 자동 생성

### 3.1 DB 스키마 자동 생성 설정 정보

```xml
<property name = "hibernate.hbm2ddl.auto" value = "create"/>

- create: DROP + CREATE
- create-drop: DROP + CREATE + DROP
- update: 테이블과 엔티티 매핑정보를 비교하여 변경 사항만 수정
- validate: 테이블과 엔티티 매핑정보를 비교하여 차이가 있으면 경고를 남기고 애플리케이션 실행을 실행하지 않는다.
- none: 자동 생성 기능을 사용하지 않는다.

* 주의사항
- 개발 초기 단계는 create 또는 update를 사용
- 초기화 상태로 자동화된 테스트를 진행하는 개발자 환경과 CI 서버는 create 또는 create-drop을 사용
- 테스트 서버는 update 또는 validate 사용
- 스테이징과 운영 서버는 validate 또는 none을 사용

* 이름 매핑 전략 설정
테이블 명이나 컬럼 명이 생략되면 자바의 카멜 표기법을 테이블의 언더스코어 표기법으로 매핑
<property name = "hibernate.ejb.naming_strategy" value = "org.hibernate.cfg.ImproveNamingStrategy"/>
```

## 4. DDL 생성 기능
- 이런 기능들은 DDL을 자동 생성할 때만 사용되고 JPA의 실행 로직에는 영향을 주지 않는다.
```java
@Entity(name = "Member")
@Table(name = "MEMBER", uniqueConstraints = {@UniqueConstraint(
    name = "NAME_AGE_UNIQUE",
    columNames = {"NAME", "AGE"} )})
public class Member {

    @Id
    @Column(name = "id")
    private String id;

    @Column(name = "name")
    private String username;

    private Integer age;
}
```
```sql
-- sql 생성
create table MEMBER (
    ID varchar(255) not null,
    NAME varchar(10) not null,
    ...
    primary key (ID),
    constraint NAME_AGE_UNIQUE unique (NAME, AGE)
)
```

## 5. 기본 키 매핑

### 5.1 키 생성 전략 설정 정보
```xml
<property name = "hibernate.id.new_generator_mappings" value="true"/>
```

### 5.2 기본 키 직접 할당 전략 (비추)
`em.persist()`로 엔티티를 저장하기 전에 애플리케이션에서 기본 키를 직접 할당하는 전략

```java
@Id
private String id;
```

### 5.3 IDENTITY 전략
기본 키 생성을 DB에 위임하는 전략 (쓰기 지연 X)

```java
@Entity
public class Board {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
}
```
```sql
--sql 생성
create table board (
    id int not null auto_increment primary key,
    data varchar(255)
)
```

#### 과정
엔티티를 영속성 컨텍스트에 저장하기 위해선 식별자 값인 id가 있어야한다. 하지만 IDENTITY 전략은 기본 키 생성을 DB에 위임하기 때문에 DB에서 식별자 값을 가져와서 엔티티에 저장 후 영속성 컨텍스트에 저장해야한다.

- em.persist(board) -> insert 쿼리 실행 -> PK 생성 및 DB에 데이터 저장 후 PK 반환 -> PK를 식별자로 board를 1번 캐시에 저장
- JDBC3에 추가된 Statement.getGeneratedKeys()를 사용함으로써 데이터를 저장하면서 동시에 생성된 기본키 값을 얻어옴으로써 DB와 한 번만 통신한다.

### 5.4 SEQUENCE 전략
시퀀스를 사용해서 기본 키를 생성하는 전략 (쓰기 지연 O)

```java
@Entity
@SequenceGenerator(
    name = "BOARD_SEQ_GENERATOR",
    sequenceName = "BOARD_SEQ",
    initialValue = 1, allocationSize = 1)
public class Board {
    @Id @GeneratedValue(strategy = GenerationType.SEQUENCE,
                        generator = "BOARD_SEQ_GENERATOR")
    private Long id;
}
```
```sql
--sql 생성
create table board (
    id bigint not null primary key,
    data varchar(255)
)

create sequence BOARD_SEQ start with 1 increment by 1;
```
#### 과정
- em.persist(board) -> 시퀀스를 사용해서 식별자 조회 -> 조회한 식별자를 엔티티에 저장 -> 엔티티를 영속성 컨텍스트에 저장 -> flush() -> DB에 반영

#### @SequenceGenerator 기능
- `@SequenceGenerator.name`: 식별자 생성기 이름 지정 (필수)
- `@SequenceGenerator.sequenceName`: DB에 등록된 시퀀스 이름 지정 (기본값: hibernate_sequence)
- `@SequenceGenerator.initialValue`: 시퀀스 DDL 생성할 때 처음 시작하는 수 지정 (기본값: 1)
- `@SequenceGenerator.allocationSize`: 시퀀스 한 번 호출에 증가하는 수 지정 (기본 값: 50)
- `@SequenceGenerator.catalog, schema`: DB catelog, schema 이름 지정

#### 최적화
JPA는 시퀀스에 접근하는 횟수를 줄이기 위해 allocationSize를 사용한다.
- allocationSize로 설정한 값만큼 한 번에 시퀀스 값을 증가시키고 그만큼 메모리에 시퀀스 값을 할당
- 시퀀스 값을 가져올 때 할당한 만큼은 메모리에서 가져온다.

### 5.5 TABLE 전략 (비추)
키 생성 전용 테이블을 하나 만들고 시퀀스를 흉내내는 전략

```java
@Entity
@TableGenerator (
    name = "BOARD_SEQ_GENERATOR",
    table = "MY_SEQUENCE",
    pkColumnValue = "BOARD_SEQ", allocationSize = 1)
public class Board {
    @Id @GeneratedValue(strategy = GenerationType.TABLE,
                        generator = "BOARD_SEQ_GENERATOR")
    private Long id;
}
```
```sql
--sql 생성
create table MY_SEQUENCES (
    sequence_name varchar(255) not null,
    next_val bigint,
    primary key (sequence_name)
)
```

### 5.6 AUTO 전략
선택한 DB 방언에 따라 3개의 전략 중 하나를 자동을 선택 (strategy 생략시 AUTO 전략 적용)

```java
@Entity
public class Board {
    @Id @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
}
```

### 5.7 권장하는 기본 키 생성 전략
- Long 타입 + 대리 키 + IDENTITY or SEQUENCE

## 6. @Column
객체 필드를 테이블 컬럼에 매핑

### 6.1 자주 사용하는 @Column 속성
- `@Column.name`: 필드와 매핑할 테이블의 컬럼 이름 지정 (기본값: 객체의 필드 명)
- `@Column.nullable`: null 값을 허용하는지 여부 (기본값: true)
- `@Column.length`: 문자 길이 제약 조건 (기본값: 255, String만 사용 가능)

```java
@Column(name = "MEMBER", nullable = false, length = 400)
private String data;
```
## 7. @Enumerated
자바의 enum 타입을 매핑

### 7.1 @Enumerated 속성
- `EnumType.ORIDINAL`: enum 순서를 DB에 저장 (기본값)
- `EnumType.STRING`: enum 이름을 DB에 저장

```java
enum RoleType {
    ADMIN, USER
}

@Enumerated(EnumType.STRING)
private RoleType roleType;
```
## 8. @Temporal
날짜 타입을 매핑

### 8.1 @Temporal 속성
- `TemporalType.DATE`: date 타입과 매핑
- `TemporalType.TIME`: time 타입과 매핑
- `TemporalType.TIMESTAMP`: timestamp 타입과 매핑
- 자바 8의 LocalDate와 LocalDateTime을 사용하면 @Temporal을 생략해도 hibernate가 자동으로 date, timestamp 타입으로 매핑해준다.

```java
@Temporal(TemporalType.TIMESTAMP) // 생략 가능
private LocalDate localDateTime;
```
## 9. @Lob
BLOB, CLOB 타입을 매핑
- `CLOB`: String, char[], java.sql.CLOB
- `BLOB`: byte[], java.sql.BLOB
- 길이가 긴 데이터를 저장할 때 사용

```java
@Lob
private String lobString;

@Lob
private byte[] lobByte;
```
## 10. @Transient
특정 필드를 DB에 매핑하지 않도록 한다.
- 객체에 임시로 어떤 값을 보관하고 싶을 때 사용

```java
@Transient
private Integer temp;
```
## 11. @Access
JPA가 엔티티에 접근하는 방식을 지정
- `AccessType.FIELD`: 필드 접근 (엔티티에 존재하는 필드들을 매핑)
- `AccessType.PROPERTY`: 프로퍼티 접근 (엔티티에 존재하는 Getter 메서드를 매핑)
- @Access를 설정하지 않으면 @Id의 위치를 기준으로 접근 방식이 설정된다.

```java
// 기본으로 필드 접근을 하고 getFullName()만 프로퍼티 접근
@Entity
@Access(AccessType.FIELD)
public class Member {

    @Id
    private String id;

    @Transient
    private String firstName;

    @Transient
    private String lastName;

    @Access(AccessType.PROPERTY)
    public String getFullName() { // FULLNAME 컬럼에 firstName + lastName의 결과를 저장
        return firstName + lastName;
    }
}
```

## Reference
> 자바 ORM 표준 JPA 프로그래밍 [김영한]

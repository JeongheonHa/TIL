# JPA 소개

## 1. 객체와 RDB 사이의 패러다임 불일치 문제 해결
- `상속`: 객체는 상속이 있지만 RDB는 상속이 없다.
- `연관관계`: 객체는 참조를 사용하지만 테이블은 외래 키를 사용한다.
- `객체 그래프 탐색`: JPA는 지연로딩을 통해 탐색 범위를 지정하지 않아도 되지만 SQL은 처음 실행한 SQL에 따라 탐색 범위가 결정된다.
- `비교`: DB에서 같은 데이터를 꺼내도 새로운 객체를 생성해서 대입하는 것이기 때문에 동일성을 보장하지 않지만 JPA는 동일성을 보장한다.

## 2. JPA 란?
- `JPA`: 자바 진영의 ORM 표준 인터페이스
- `ORM`: 객체와 RDB를 매핑한다는 의미
- `Hibernate`: ORM 구현체 (대부분 Hibernate를 사용)

### 2.1 JPA 위치
애플리케이션과 JDBC 사이에서 동작한다.

<img src="https://user-images.githubusercontent.com/108064146/231292267-a5f61f33-457e-477f-bdca-e3ffdc04f49c.png">

### 2.2 JPA 설정 정보
- JPA는 persistence.xml을 사용해서 설정 정보를 관리한다.
- META-INF/persistence.xml 클래스 패스 경로에 있으면 별도의 설정 없이 인식 가능

```xml
// JPA 표준 속성 
<property name = "javax.persistence.jdbc.driver" value = "org.h2.Driver"/>
<property name = "javax.persistence.jdbc.user" value = "sa"/>
<property name = "javax.persistence.jdbc.password" value = ""/>
<property name = "javax.persistence.jdbc.url" value = "jdbc:h2:tcp://localhost/~/test"/>

// Hibernate 속성
<property name = "hibernate.dialect" value = "org.hibernate.dialect.H2Dialect"/> // 방언 설정

// 방언들
org.hibernate.dialect.H2Dialect           // H2
org.hibernate.dialect.Oracle10gDialect    // 오라클 10g
org.hibernate.dialect.MySQL5InnoDBDialect // MySQL

// Hibernate 추가 속성 (value = true / false)
hibernate.show_sql                  // sql 출력
hibernate.format_sql                // sql 출력 정렬
hibernate.use_sql_comments          // 쿼리 출력시 주석도 함께 출력
hibernate.id.new_generator_mappings // 키 생성 전략 
```

## 3. JPA 시작

### 3.1 EntityManagerFactory 생성
- J2SE 환경에서 Hibernate를 포함한 JPA 구현체들은 EntityManagerFactory를 생성할 때 커넥션 풀도 생성한다.
- J2EE 환경에서는 해당 컨테이너가 제공하는 DataSource를 사용한다. 
- 애플리케이션 전체에 하나만 생성하고 여러 쓰레드가 공유해서 사용

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("설정 정보");
```

### 3.2 EntityManager 생성
- CRUD 기능을 제공
- DB 커넥션을 유지하면서 DB와 통신하기 때문에 쓰레드간에 공유하거나 재사용하면 안된다.
```java
EntityManager em = emf.createEntityManager();
```

### 3.3 트랜잭션 관리
JPA를 사용하면 항상 트랜잭션 안에서 데이터를 변경해야한다. (트랜잭션 없이 데이터를 변경할 경우 예외 발생)

```java
EntityTransaction tx = em.getTransaction(); // 트랜잭션 API

try {
    tx.begin();     // 트랜잭션 시작
    bizLogic(em);   // 비즈니스 로직 실행
    tx.commit();    // 커밋
} catch (Exception e) {
    tx.rollback();  // 롤백
}
```
### 3.4 등록, 수정, 삭제, 조회

```java
- em.persist(member); // 등록
- member.setAge(10);  // 수정
- em.remove(member);  // 삭제
- Member findMember = em.find(Member.class, id); // 조회
```

### 3.5 JPQL
- `JPQL`: 엔티티 객체를 대상으로 쿼리 (대소문자 구분 O)
- `SQL`: DB 테이블을 대상으로 쿼리 (대소문자 구분 X)
- JPQL은 DB 테이블을 알지 못한다.
```java
TypeQuery<Member> query = em.createQuery("select m from Member m", Member.class);
List<Member> members = query.getResultList();
```

## Reference
> 자바 ORM 표준 JPA 프로그래밍 [김영한]

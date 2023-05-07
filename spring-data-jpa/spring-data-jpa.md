# Spring Data JPA

## 1. 소개

## 1.1 스프링 데이터 JPA 란

스프링 데이터 JPA는 스프링 프레임워크에서 JPA를 편리하게 사용할 수 있도록 지원하는 프로젝트이다.

- 스프링 데이터 JPA는 스프링 데이터 프로젝트의 하위 프로젝트 중 하나이다. 
- 스프링 데이터 프로젝트는 JPA, 몽고 DB, redis, hadoop 등 다양한 데이터 저장소에 대한 접근 을 추상화해서 인터페이스로 제공한다.

### 1.2 의존 관계 설정

```gradle
implementation ‘org.springframework.boot:spring-boot-starter-data-jpa’
```

## 2. 공통 인터페이스

### 2.1 공통 인터페이스 설정

스프링 부트 사용시 자동 설정된다.

```java
// Config 설정 객체 사용시
@Configuration
@EnableJpaRepositories(basePackages = "jpabook.jpashop.repository")
public class AppConfig {}
```

```java
// 스프링 부트 사용시
@EnableJpaRepositories(basePackages = "jpabook.jpashop.repository") // 생략 가능
@SpringBootApplication
public class DataJpaApplication {}
```


### 2.2 공통 인터페이스 적용

```java
// JpaRepository<엔티티 타입, 식별자 타입(PK)>
public interface MemberRepository extends JpaRepository<Member, Long> {
}
```

- 리포지토리 개발시 org.springframework.data.repository.Repository를 상속받는 인터페이스만 작성하면 실행 시점에 스프링 데이터 JPA가 구현 객체를 동적으로 생성해서 스프링 빈으로 등록해준다.
- 이렇게 생성된 객체는 프록시 객체로 생성되어 스캔의 대상이 된다.
- @Repository 애노테이션 생략 가능
    + 컴포넌트 스캔을 스프링 데이터 JPA가 자동으로 처리
    + JPA 예외를 스프링 예외로 변환하는 과정도 자동으로 처리


### 2.3 공통 인터페이스 구성

<p align="center">
<img src="https://user-images.githubusercontent.com/108064146/236140550-b44b59c8-5a49-4ec9-9c5b-3e2af60d2096.png" style=" margin: 0 auto;width:500px; height:700px;"
/>
</p>

#### 주요 메서드

```html
// T: 엔티티, ID: 식별자, S: 엔티티와 그 자식 타입
- save(S): 새로운 엔티티는 저장하고 이미 있는 엔티티는 병합
- delete(T): 엔티티 하나 제거
- findById(ID): 식별자로 엔티티 하나 조회
- getOne(ID): 엔티티를 프록시로 조회
- findAll(...): 모든 엔티티를 조회 (정렬이나 페이징 조건을 파라미터로 제공가능)
```

## 3. 쿼리 메서드 기능

- 메서드 이름으로 쿼리 생성
- 메서드 이름으로 JPA NamedQuery 호출
- @Query 애노테이션을 사용하여 리포지토리 인터페이스에 쿼리 직접 정의

### 3.1 메서드 이름으로 쿼리 생성
스프링 데이터 JPA는 메서드의 이름을 분석해 JPQL 쿼리를 실행한다.

- 필드명 변경시 해당 메서드 명도 수정해야함.(필드명과 메서드 명이 다르면 app 로딩 시점에 오류 발생)

```java
// 구현체 없이 메서드 선언부 만으로 기능 수행
public interface MemberRepository extends JpaRepository<Member, Long> {
    List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
}
```
|키워드|예|JPQL 예|
|:---|:---|:---|
|Distinct|findDistinctByLastnameAndFirstname|select distinct …​ where x.lastname = ?1 and x.firstname = ?2|
|And|findByLastnameAndFirstname|… where x.lastname = ?1 and x.firstname = ?2|
|Or|findByLastnameOrFirstname|… where x.lastname = ?1 or x.firstname = ?2|
|Is, Equals|findByFirstname,findByFirstnameIs,<br>findByFirstnameEquals|… where x.firstname = ?1|
|Between|findByStartDateBetween|… where x.startDate between ?1 and ?2|
|LessThan|findByAgeLessThan|… where x.age < ?1|
|LessThanEqual|findByAgeLessThanEqual|… where x.age <= ?1|
|GreaterThan|findByAgeGreaterThan|… where x.age > ?1|
|GreaterThanEqual|findByAgeGreaterThanEqual|… where x.age >= ?1|
|After|findByStartDateAfter|… where x.startDate > ?1|
|Before|findByStartDateBefore|… where x.startDate < ?1|
|IsNull, Null|findByAge(Is)Null|… where x.age is null|
|IsNotNull, NotNull|findByAge(Is)NotNull|… where x.age not null|
|Like|findByFirstnameLike|… where x.firstname like ?1|
|NotLike|findByFirstnameNotLike|… where x.firstname not like ?1|
|StartingWith|findByFirstnameStartingWith|… where x.firstname like ?1 (parameter bound with appended %)|
|EndingWith|findByFirstnameEndingWith|… where x.firstname like ?1 (parameter bound with prepended %)|
|Containing|findByFirstnameContaining|… where x.firstname like ?1 (parameter bound wrapped in %)|
|OrderBy|findByAgeOrderByLastnameDesc|… where x.age = ?1 order by x.lastname desc|
|Not|findByLastnameNot|… where x.lastname <> ?1|
|In|findByAgeIn(Collection<Age> ages)|… where x.age in ?1|
|NotIn|findByAgeNotIn(Collection<Age> ages)|… where x.age not in ?1|
|True|findByActiveTrue()|… where x.active = true|
|False|findByActiveFalse()|… where x.active = false|
|IgnoreCase|findByFirstnameIgnoreCase|… where UPPER(x.firstname) = UPPER(?1)|
>출처: <https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation>

### 3.2 JPA NamedQuery

쿼리에 이름을 부여해서 사용하는 방식

- @Query 생략시 스프링 데이터 JPA는 선언한 `도메인 클래스 + . + 메서드 이름`으로 NamedQuery를 찾아서 실행 (JpqRepository<Member, Long>)
- 실행할 NamedQuery가 없으면 메서드 이름으로 쿼리 생성 전략 사용
- **실행 시점에 문법 오류 발견 가능**
- 쿼리를 사용하는 건 리포지토리인데 도메인에 NamedQuery를 지정해야하는 불편함 존재
- 해당 방식보다 리포지토리 메서드에 쿼리를 지정하는 방식을 사용

```java
@Entity
@NamedQuery(
    name="Member.findByUsername",
    query="select m from Member m where m.username = :username")
public class Member {
    //... 
}
```

```java
public interface MemberRepository extends JpaRepository<Member, Long> { //** 여기 선언한 Member 도메인 클래스

    @Query(name = "Member.findByUsername") // 생략하고 메서드 이름만으로 NamedQuery 호출 가능
    List<Member> findByUsername(@Param("username") String username);
}
```

### 3.3 리포지토리 메서드에 쿼리 정의
실행할 메서드에 정적 쿼리를 직접 작성하는 방식으로 `이름 없는 NamedQuery`라고 할 수 있다.

- **실행 시점에 문법 오류 발견 가능**

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query("select m from Member m where m.username= :username and m.age = :age")
    List<Member> findUser(@Param("username") String username, @Param("age") int age);
}
```

#### DTO로 직접 조회할 경우

```java
@Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) " + "from Member m join m.team t")
List<MemberDto> findMemberDto();
```

- JPA 값 타입도 해당 방식으로 조회 가능

## 4. 파라미터 바인딩

### 4.1 이름 기반 파라미터 바인딩
`위치 기반(?0)`과 `이름 기반(:name)`이 있으며 이름 기반을 사용해야한다. (위치 기반은 실수할 우려가 많음.)

```java
@Query("select m from Member m where m.username = :name")
Member findMembers(@Param("name") String username);
```

### 4.2 컬렉션 파라미터 바인딩
Collection 타입으로 `in 절`을 지원한다.

```java
@Query("select m from Member m where m.username in :names")
List<Member> findByNames(@Param("names") List<String> names);
```

## 5. 반환 타입

스프링 데이터 JPA는 유연한 반환 타입을 지원한다.

```java
List<Member> findByUsername(String name); //컬렉션 
Member findByUsername(String name); //단건
Optional<Member> findByUsername(String name); //단건 Optional
```

### 5.1 반환 타입의 데이터가 없거나 더 많은 경우

```html
- 컬렉션
    + 결과 X: 빈 컬렉션 반환
- 단건 조회
    + 결과 X: null 반환
    + 결과 2건 이상: javax.persistence.NonUniqueResultException예외 발생
```

- 단건 조회의 경우 내부적으로 `Query.getSingleResult()`를 호출하기 때문에 결과 값이 없다면 `javax.persistence.NoResultException`예외를 발생시킨다. 하지만 스프링 데이터 JPA가 예외를 무시하고 null로 반환해준다.
- 가장 간단한 방법은 값이 없을 수도 있는 경우 Optional<T>로 반환하여 null 발생시 처리를 클라이언트 쪽으로 넘기는 것이다.

> 추가 반환 타입에 대한 공식 문서: <https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#appendix.query.return.types>

## 6. 페이징과 정렬
스프링 데이터 JPA는 쿼리 메서드에 페이징과 정렬 기능을 사용할 수 있도록 2가지 특별한 파라미터를 제공한다.

### 6.1 페이징과 정렬 파라미터

- `org.springframework.data.domain.Sort`     : 정렬 기능
- `org.springframework.data.domain.Pageable` : 페이징 기능 (내부에 Sort포함)

### 6.2 페이징 반환 타입
- `org.springframework.data.domain.Page`  : 추가 count 쿼리 결과를 포함하는 페이징
- `org.springframework.data.domain.Slice` : 추가 count 쿼리 없이 다음 페이지만 확인 가능 (내부적으로 limit + 1 조회)
- `List(자바 컬렉션)` : 추가 count 쿼리 없이 결과만 반환


### 6.3 페이징과 정렬 사용

- 페이지는 0부터 시작한다.

#### 메서드 정의
```java
// 리포지토리에 메서드 정의
public interface MemberRepository extends Repository<Member, Long> {
    Page<Member> findByUsername(String name, Pageable pageable);  // count 쿼리 사용
    Slice<Member> findByUsername(String name, Pageable pageable); // count 쿼리 사용 안함
    List<Member> findByUsername(String name, Pageable pageable);  // count 쿼리 사용 안함
    List<Member> findByUsername(String name, Sort sort);
}
```

#### 페이징, 정렬 사용
- `PageRequest`: Pageable 인터페이스를 구현한 객체 (`.of(현재 페이지, 조회할 데이터 수, 정렬 정보)`)

```java
PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));
Page<Member> page = memberRepository.findByAge(10, pageRequest); 

List<Member> content = page.getContent();
```

### 6.4 Page, Slice 인터페이스

- `Slice`는 `limit + 1` 개의 페이지나 데이터를 가져와 다음이 있는지 없는지만을 확인하기 위한 인터페이스이다.
    + ex. 다음 페이지가 있으면 `더보기`
    + ex. 다음 페이지가 없으면 `마지막 페이지`

- `Page`는 전체 페이지나 데이터를 가져오는 기능을 추가로 갖고있는 인터페이스이다. (Slice 상속)

#### Page 인터페이스

```java
public interface Page<T> extends Slice<T> {
    int getTotalPages(); //전체 페이지 수
    long getTotalElements(); //전체 데이터 수
    <U> Page<U> map(Function<? super T, ? extends U> converter); //변환기 (DTO 변환시 유용)
}
```

#### Slice 인터페이스

```java
public interface Slice<T> extends Streamable<T> {
    int getNumber();            // 현재 페이지
    int getSize();              // 페이지 크기
    int getNumberOfElements();  // 현재 페이지에 나올 데이터 수
    List<T> getContent();       // 조회된 데이터
    boolean hasContent();       // 조회된 데이터 존재 여부
    Sort getSort();             // 정렬 정보
    boolean isFirst();          // 현재 페이지가 첫 페이지인지 여부
    boolean isLast();           // 현재 페이지가 마지막 페이지인지 여부
    boolean hasNext();          // 다음 페이지 여부
    boolean hasPrevious();      // 이전 페이지 여부
    Pageable getPageable();     // 현재 페이지 객체
    Pageable nextPageable();    // 다음 페이지 객체
    Pageable previousPageable();//이전 페이지 객체
    <U> Slice<U> map(Function<? super T, ? extends U> converter); //변환기 (DTO 변환시 유용)
}
```

#### Slice와 Page 쿼리 비교
- offest은 0부터 시작이기 때문에 생략
- Slice는 limit + 1개의 데이터를 Limit로 날려 데이터를 가져오지만 Page는 count()를 이용해 데이터를 가져오기 때문에 성능상 Slice가 우위에 있을 수 밖에 없다.

```sql
-- Slice
select 
    id, age, username
from
    member
where 
    age = ?
order by
    username desc limit ? 
```

```sql
-- Page
select
    count(id)
from 
    member
where
    age = ?
```

### 6.5 count 쿼리 분리
count 쿼리는 DB의 모든 데이터를 세야하기 때문에 count라는 함수 자체가 성능을 저하시킨다. 따라서, 단순히 데이터 개수를 알고 싶다면, 굳이 join과 함께 count를 할 필요가 없다. 그냥 조인을 통해 데이터를 가져오는 퀴리 따로 count 쿼리 따로 가져오면 된다.

```java
@Query(value = “select m from Member m”, 
       countQuery = “select count(m.username) from Member m”)
Page<Member> findMemberAllCountBy(Pageable pageable);
```

### 6.6 Top, First 사용
메서드 명 방식으로 가져올 수 도 있다.

```java
User findFirstByOrderByLastnameAsc();

User findTopByOrderByAgeDesc();

Page<User> queryFirst10ByLastname(String lastname, Pageable pageable);

Slice<User> findTop3ByLastname(String lastname, Pageable pageable);

List<User> findFirst10ByLastname(String lastname, Sort sort);

List<User> findTop10ByLastname(String lastname, Pageable pageable);
```

### 6.7 페이지를 유지하면서 엔티티를 DTO로 변환
- Page나 Slice 인터페이스가 제공하는 map() 변환기를 사용하면 Page 내부의 엔티티를 DTO로 변환해서 반환해준다. 개발자는 DTO를 감싼 Page 객체를 api로 제공하기만 하면 된다. (JSON으로 변환됨.)
```java
Page<Member> page = memberRepository.findByAge(10, pageRequest);
Page<MemberDto> dtoPage = page.map(m -> new MemberDto());
```

## 7. 벌크성 쿼리
- 벌크성 수정, 삭제 쿼리는 `@Modifying`애노테이션을 꼭 붙여야한다.
    + @Modifying이 붙어있어야 executeUpdate()를 실행한다.
    + @Modifying이 없다면 아래와 같은 예외를 발생시킨다.

```
// @Modifying 애노테이션을 붙이지 않을 경우 해당 예외 발생
org.hibernate.hql.internal.QueryExecutionRequestException: Not supported for DML operations
```

- 반환 값은 변경된 데이터의 갯수를 반환하고 반환 타입은 `void`, `int`, `Integer`만 가능하다.
- 스프링 데이터 JPA는 벌크성 쿼리 실행 후 영속성 컨텍스트를 초기화하는 `@Modifying(clearAutomatically = true)`를 제공한다.

```java
@Modifying(clearAutomatically = true)
@Query("update Member m set m.age = m.age + 1 where m.age >= :age")
int bulkAgePlus(@Param("age") int age);
```

- 벌크성 쿼리는 영속성 컨텍스트를 무시하고 쿼리를 날리기 때문에 영속성 컨텍스트의 엔티티와 DB에 있는 실제 엔티티의 상태가 달라질 수 있다. 
    + 해결 1: 영속성 컨텍스트에 엔티티가 있다면 벌크성 쿼리를 날린 직 후에 바로 영속성 컨텍스트를 초기화
    + 해결 2: 영속성 컨텍스트에 엔티티가 없는 상태에서 벌크성 쿼리 실행

## 8. 엔티티 그래프
연관된 엔티티들을 SQL 한번에 조회하는 방법
- 페치 조인의 간편 버전
- LEFT OUTER JOIN 사용

### 8.1 사용 방법

```java
//공통 메서드 오버라이드
@Override
@EntityGraph(attributePaths = {"team"}) 
List<Member> findAll();

//JPQL + 엔티티 그래프
@EntityGraph(attributePaths = {"team"}) 
@Query("select m from Member m") 
List<Member> findMemberEntityGraph();

//메서드 이름으로 쿼리에서 특히 편리하다.
@EntityGraph(attributePaths = {"team"}) 
List<Member> findByUsername(String username)
```

## 9. JPA Hint & Lock

### 9.1 JPA Hint
JPA 쿼리 힌트 (SQL 힌트가 아닌 JPA 구현체에게 제공하는 힌트)
- `org.springframework.data.jpa.repository.QueryHints` 애노테이션 사용
- 하이버네이트는 제공하지만 JPA는 제공하지 않는 기능들을 사용하고자 할 때 힌트를 제공해서 사용하도록 할 수 있다.
- 주로 데이터를 읽기만 할 때 스냅샷을 생성하지 않고 변경 감지를 생략하고자 할 때 사용한다.

```java
@QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value = "true"))
Member findReadOnlyByUsername(String username);
```

```java
@Test
public void queryHint() throws Exception {
    //given
    memberRepository.save(new Member("member1", 10));

    em.flush();
    em.clear();

    //when
    Member member = memberRepository.findReadOnlyByUsername("member1");

    member.setUsername("member2");

    em.flush(); //Update Query 실행X 
}
```

- `사용시 주의 사항` : 성능에 문제가 있는 경우는 보통 복잡한 조회 쿼리를 잘못짜서 발생하는 경우가 많기 때문에 조회 쿼리를 잘 짰는데도 성능 최적화가 필요한 경우 사용하도록 하자.

### 9.2 Lock
쿼리에 락을 걸고 싶을 때 사용한다. (ex. select ... for update)
- `org.springframework.data.jpa.repository.Lock` 애노테이션 사용
- 참고로 실시간 트래픽이 많은 경우 Lock을 걸면 안된다.

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
List<Member> findByUsername(String name);
```


## Reference
> 실전! 스프링 데이터 JPA [김영한]

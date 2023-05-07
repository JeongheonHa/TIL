# 확장 기능

## 1. 사용자 정의 리포지토리 구현
리포지토리 인터페이스에 정의된 메서드만이 아닌 메서드를 직접 구현해야 하는 경우 사용자 정의 리포지토리를 이용해서 해결할 수 있다.

- 스프링 데이터 JPA가 제공하는 기술로 Java에서는 안된다.

- 사용자 정의 인터페이스를 구현한 구현체의 이름은 아래 둘 중 하나여아한다.

    + `리포지토리 인터페이스 이름 + Impl`
    + `사용자 정의 인터페이스 이름 + Impl`

- 주로 `QueryDSL`, `Jdbc Template` 등을 사용하기 위해 사용

<img src = "https://user-images.githubusercontent.com/108064146/236673705-24568746-ea43-42af-a9d9-0260de8c1e83.JPG">

```java
// 사용자 정의 인터페이스 정의
public interface MemberRepositoryCustom {
    public List<Member> findMemberCustom();
}

// 사용자 정의 인터페이스 구현
public class MemberRepositoryImpl implements MemberRepositoryCustom {

    @Override
    public List<Member> findMemberCustom() {
        // 구현
    }
}

// 사용자 정의 인터페이스 상속
public interface MemberRepository extends JpaRepository<Member, Long> MemberRepositoryCustom {
}
```

### 1.1 언제 사용할까?
1. 리포지토리 내장 메서드로 해결
2. 메서드 이름을 쿼리를 생성하는 것으로 해결
3. 리포지토리에 @Query를 사용하는 것으로 해결
4. 사용자 정의 인터페이스를 사용해서 해결

### 1.2 주의사항
항상 사용자 정의 리포지토리가 필요한 것은 아니다.
- 예를 들어, api 스펙이 맞춰서 DTO로 데이터를 가져올 경우 쿼리가 복잡해진다. 이 때 사용자 정의 함수를 사용한다면 MemberRepository 인터페이스에 상속되기 때문에 복잡한 쿼리와 단순한 쿼리가 섞여서 구분하기 힘들게 된다. 따라서, MemberQueryRepository 클래스를 따로 스프링 빈으로 등록하여 command와 query를 분리하는 것이 좋다.

## 2. Auditing
엔티티를 생성, 변경할 때 변경한 사람과 시간을 추적하고 싶을 때 사용.
- `등록일`, `수정일`, `등록자`, `수정자`

### 2.1 순수 JPA 사용

```
- `@PrePersist`: 저장하기 전에 실행
- `@PostPersist`: 저장한 후에 실행
- `@PreUpdate`: 변경하기 전에 실행
- `@PostUpdate`: 변경한 후에 실행
```

```java
@MappedSuperclass
@Getter
public class JpaBaseEntity {

    @Column(updateable = false)
    private LocalDateTime createdDate;
    private LocalDateTime updatedDate;

    @PrePersist
    public void prePersist() {
        LocalDateTime now = LocalDateTime.now();
        createdDate = now;
        updatedDate = now;
    }

    @PreUpdate
    public void preUpdate() {
        udatedDate = LocalDateTime.now();
    }
}

public class Member extends JpaBaseEntity {
}
```

### 2.2 Auditing 사용

```
// 스프링 데이터 JPA가 제공
- `@CreatedDate`: 생성일 지정
- `@LastModifedDate`: 수정일 지정
- `@CreateBy`: 생성자 지정
- `@LastModifedBy`: 수정자 지정
```

- `EnableJpaAuditing`을 스프링 부트 설정 클래스에 적용해야한다.
- `@EntityListeners(AuditingEntityListener.class)`를 엔티티에 적용해야 한다.

```java
@EnableJpaAudting
@SpringBootApplication
public class DataJpaApplication {
    // ...
}
```

- 보통 등록일과 수정일은 모든 엔티티에 필요하지만 등록자와 수정자는 필요하지 않을 수 있기 때문에 상속을 통해 구분해주는 것이 좋다.

```java
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
public class BaseTimeEntity {

    @CreatedDate
    @Column(updatable = false) // 변경 못하게 설정
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime lastModifiedDate;
}

public class BaseEntity extends BaseTimeEntity {

    @CreatedBy
    @Column(updateable = false) // 변경 못하게 설정
    private String createdBy;

    @LastModifiedBy
    private String lastModifiedBy;
}
```

- 등록자, 수정자를 처리해주는 AuditorAware를 스프링 빈으로 등록
- 실무에서는 세선 정보나, 스프링 시큐리티 로그인 정보에서 ID를 받는다.

```java
@EnableJpaAudting
@SpringBootApplication
public class DataJpaApplication {
    // ...

    @Bean
    public AuditorAware<String> auditorProvider() {
        return // ID를 반환
    }
}
```
- 저장 시점에 등록일, 등록자와 같은 값을 수정일, 수정자에 넣는 것은 데이터가 중복되어 안좋은 방법으로 보일 수 있지만, 변경 컬럼만 확인해도 마지막에 업데이트한 시간이나 유저를 확인할 수 있으므로 유지 보수 관점에서 편리하다.
- 만약, null로 하고싶다면 null로 저장하고 싶은 필드에 `@EnableJpaAuditing(modifyOnCreate = false)`로 설정하면 된다.

### 2.3 @EntityListeners 전체 적용
해당 xml 파일을 META-INF/orm.xml에 설정해주면 JPA가 제공하는 이벤트를 엔티티 전체에 적용할 수 있다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
    <entity-mappings xmlns="http://xmlns.jcp.org/xml/ns/persistence/orm"
                     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                     xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence/orm http://xmlns.jcp.org/xml/ns/persistence/orm_2_2.xsd" 
                     version="2.2">
        <persistence-unit-metadata>
            <persistence-unit-defaults>
                        <entity-listeners>
                            <entity-listener class="org.springframework.data.jpa.domain.support.AuditingEntityListener"/>
                        </entity-listeners>
            </persistence-unit-defaults>
        </persistence-unit-metadata>
    </entity-mappings>
```

## 3. Web 확장 - 도메인 클래스 컨버터 기능
도메인 클래스 컨버터는 HTTP 파라미터로 넘어온 엔티티의 아이디로 엔티티 객체를 찾아서 바인딩해준다.
- 도메인 클래스 컨버터도 Repository를 사용해서 엔티티를 찾는다.

```java
// 도메인 클래스 컨버터가 id를 회원 엔티티로 변환
@GetMapping("/members/{id}")
public String findMember(@PathVariable("id") Member member) {
    return member.getUsername();
}
```

- 도메인 클래스 컨버터를 통해 넘어온 엔티티를 컨트롤러에서 수정해도 실제 DB에 반영되지 않는다.
    + `OSIV OFF`의 경우: 영속성 컨텍스트의 생명주기가 commit을 한 시점까지이기 때문에 해당 엔티티는 준영속 상태가 되어 변경 감지를 할 수 없다. 따라서, 변경하고 싶다면 merge()를 사용해야한다.
    + `OSIV ON`의 경우: 해당 엔티티는 영속상태이지만 컨트롤러와 view에서는 영속성 컨텍스트를 flush하지 않기 때문에 엔티티를 수정해도 DB에 반영되지 않는다.
    + 따라서, 도메인 클래스 컨버터로 엔티티를 파라미터로 받으면, 해당 엔티티는 단순 조회용으로만 사용해야한다.

- 실무에서는 이렇게 단순한 쿼리는 보통 없기 때문에 사용을 권장하지 않는다.

## 4. Web 확장 - 페이징, 정렬 리졸버
스프링 데이터가 제공하는 페이징과 정렬 기능을 스프링 MVC에서 편리하게 사용할 수 있도록 `HandlerMethodArgumentResolver`를 제공

- `페이징 기능`: PageableHandlerMethodArgumentResolver
- `정렬 기능`: SortHandlerMethodArgumentResolver
- `Pageable`: Pageable은 인터페이스이기 때문에 org.springframework.data.domain.PageRequest 객체가 생성되어 제공된다.

```java
@GetMapping("/members")
public Page<MemberDto> list(Pageable pageable) {
    return memberRepository.findAll(pageable)
            .map(member -> new MemberDto(member.getId(), member.getUsername(), null));
}
```

- `요청`: /members?page=0&size=3&sort=id,desc&sort=username,desc
- `page`: 현재 페이지 (기본 0부터 시작)
- `size`: 한 페이지에 노출할 데이터 건수
- `sort`: 정렬 조건 정의

### 4.1 기본값 

#### 글로벌 설정
설정 파일에서 기본값을 변경할 수 있다.

```xml
spring.data.web.pageable.default-page-size=20 /# 기본 페이지 사이즈/ 
spring.data.web.pageable.max-page-size=2000 /# 최대 페이지 사이즈/
```

#### 개별 설정
`@PageableDefault`애노테이션을 사용하면 개별 설정이 가능하다.
```java
@GetMapping("members_page")
public String list(@PageableDefault(size = 12, sort = "username", direction = Sort.Direction.DESC) Pageable pageable) {
    //...
}
```

### 4.2 접두사
페이징 정보가 둘 이상이라면 접두사로 구분할 수 있다.
- `@Qualifier`에 접두사 명 추가 `"접두사명_xxx"`
- 요청 예시: `/members?member_page-0&order_page=1`

```java
public String list(
    @Qulifier("member") Pageable memberPageable,
    @Qulifier("order") Pageable orderPageable)
```

### 4.3 페이지 1부터 시작
페이지를 1부터 시작하고 싶으면 `PageableHandlerMethodArgumentResolver`를 스프링 빈으로 직접 등록하고 `setOneIndexdParameters`를 true로 설정하면 된다.

- `spring.data.web.pageable.one-indexed-parameters` 를 true 로 설정할 경우 web에서 page파라미터를 -1처리 할 뿐이다. 따라서 응답값인 Page에 모두 0페이지 인덱스를 사용하는 한계가 있다.


## Reference
> 실전! 스프링 데이터 JPA [김영한]

# 값 타입

## 1. 값 타입의 종류
- `기본값 타입`: 자바 기본 타입, 래퍼 클래스, String
- `임베디드 타입`
- `컬렉션 값 타입`

### 1.1 값 타입의 장점
값 타입의 생명 주기는 엔티티에 의존한다.

## 2. 기본값 타입
- 기본값 타입은 절대 공유되지 않는다.
- 생명 주기는 엔티티에 의존한다.

## 3. 임베디드 타입(복합 값 타입)
직접 정의해서 사용하는 값 타입
- `@Embeddable`: 값 타입을 정의하는 곳에 표시
- `@Embedded`: 값 타입을 사용하는 곳에 표시
- 기본 생성자 필수

<img src="https://user-images.githubusercontent.com/108064146/232186862-cbce9eb7-a756-4c44-8eef-77bde8acff1c.png">

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @Embedded Period workPeriod;
    @Embedded Address homeAddress;
}

@Embeddable
public class Period {

    LocalDateTime startLocalDateTime;
    LocalDateTime endLocalDateTime;

    public boolean isWork(LocalDateTime) {
        // 값 타입을 위한 메서드를 정의할 수도 있다.
    }
}

@Embeddable
public class Address {

    @Column(name = "city")
    private String city;
    private String street;
    private String zipcode;
}
```

### 3.1 임베디드 타입의 장점
- 재사용 가능
- 높은 응집도
- 해당 값 타입에 관한 메서드를 정의할 수 있다.
- 생명 주기는 엔티티에 의존한다.

### 3.2 한 엔티티에 같은 값 타입을 사용하는 경우
한 엔티티에 같은 값 타입을 사용하면 컬럼 명이 중복되게 된다.

- `@AttributeOverride`를 사용해서 속성명을 재정의 해야한다.

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;
    private String name;

    @Embedded 
    @AttributeOverrides({
        @AttributeOverride(name = "city", column = @Column(name = "HOME_CITY")),
        @AttributeOverride(name = "street", column = @Column(name = "HOME_STREET")),
        @AttributeOverride(name = "zipcode", column = @Column(name = "HOME_ZIPCODE"))
    })
    Address homeAddress;
    @Embedded Address companyAddress;
}
```

### 3.3 임베디드 타입과 null
임베디드 타입이 null이면 매핑한 컬럼 값 모두 null이된다.

## 4. 값 타입과 불변 객체
한 임베디드 타입을 두 엔티티가 사용한다고 할 때 참조 값을 저장하기 때문에 한 엔티티의 값만 바꾸고 싶어도 다른 엔티티도 같이 변경된다. 즉, Update 쿼리가 2개가 나간다.

```java
member1.setHomeAddress(new Address("OldCity"));
Address address = member1.getHomeAddress();

address.setCity("NewCity");
member2.setHomeAddress(address); // member1의 address도 변경됨.
```

따라서, 값 타입은 객체를 불변 객체로 만들어 애초에 값을 수정할 수 없도록 만들고 값을 변경하고 싶은 경우 새로운 객체를 생성해서 사용한다.

```java
@Embeddable
public class Address {

    private String city;

    protected Address() {}

    public Address(String city) {
        this.city = city;
    }

    public String getCity() {
        return city;
    }

    // setter X
}
```

## 5. 값 타입의 비교
값 타입은 인스턴스가 달라도 내부의 값이 같다면 같은 값으로 봐야한다.

### 5.1 기본 타입 비교
동일성 보장
```java
int a = 10;
int b = 10;

System.out.println(a == b); // true;
```

### 5.2 임베디드 타입 비교
임베디트 타입 객체에 `equals`와 `hashCode`를 오버라이딩해서 구현한다. (동등성 보장)

```java
Address a = new Address("서울시", "종로구", "1번지");
Address b = new Address("서울시", "종로구", "1번지");

System.out.println(a == b) // false;
System.out.println(a.equals(b)) // true;
```

## 6. 값 타입 컬렉션
값 타입을 저장하는 컬렉션
- 컬렉션을 별도의 테이블로 만들고 값 타입들을 기본 키로 구성하여 매핑한다.
- `@ElementCollection`, `@CollectionTable`을 이용해야 매핑
- 생명 주기는 엔티티에 의존한다. (member만 persist하면 모두 다 persist 된다.)
    + 즉, 값 타입 컬렉션은 영속성 전이 + 고아 객체 제거 기능을 필수로 가진다고 볼 수 있다.
- `@ElementCollection(fetch = FetchType.LAZY)`로 기본이 지연 로딩이다.

<img src = "https://user-images.githubusercontent.com/108064146/232204610-dca6c7eb-7ab1-4cd4-90ad-d596538ce3d2.png">

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    @Embedded
    private Address homeAddress;

    @ElementCollection
    @CollectionTable(name = "FAVORITE_FOOD", joinColumns = @JoinColumn(name = "MEMBER_ID"))
    @Column(name = "FOOD_NAME") // 내가 지정한 값 타입이 아니기 때문에 테이블의 속성 명을 지정해준다.
    private Set<String> favoriteFoods = new HashSet<>();

    @ElementCollection
    @CollectionTable(name = "ADDRESS", joinColumns = @JoinColumn(name = "MEMBER_ID"))
    private List<Address> addressHistory = new ArrayList<>();
}

@Embeddable
public class Address {

    @Column
    private String city;
    private String street;
    private String zipcode;
}
```

### 6.1 저장
```java
// 값 컬렉션 사용
Member member = new Member();

// 임베디드 값 타입 저장
member.setHomeAddress(new Address("전주", "한옥마을", "666-123"));

// 기본값 타입 컬렉션
member.getFavoriteFoods().add("치킨");
member.getFavoriteFoods().add("피자");
member.getFavoriteFoods().add("라면");

// 임베디드 값 타입 컬렉션
member.getAddressHistory().add(new Address("서울", "강남", "123-123"));
member.getAddressHistory().add(new Address("서울", "강북", "000-000"));

em.persist(member);
```

### 6.2 수정
```java
Member findMember = em.find(Member.class, 1L);

Address a = findMember.getHomeAddress();
findMember.setHomeAddress(new Address("서울", "종로", "111-111"));

findMember.getFavoriteFoods().remove("치킨");
findMember.getFavoriteFoods().add("한식");

findMember.getAddressHistory().remove(new Address("서울", "강남", "123-123"));
findMember.getAddressHistroy().add(new Address("부산", "해운대", "222-222"));

em.flush();
```

### 6.3 값 타입 컬렉션의 제약사항
값 타입은 식별자라는 개념이 없고 단순히 값들의 모음이기 때문에 값을 변경하면 DB에 저장된 원본 데이터를 찾기 어렵다.

- 값 타입 컬렉션에 보관된 값 타입들은 별도의 테이블에 보관되기 때문에 여기에 보관된 값이 변경되면 원본 데이터를 찾는 것은 어렵다.
- JPA 구현체들은 값 타입 컬렉션에 변경 사항이 발생하면, 값 타입 컬렉션이 매핑된 테이블의 연관된 모든 데이터를 삭제하고, 현재 값 타입 컬렉션 객체에 잇는 모든 값을 DB에 다시 저장한다. (즉, 1개만 변경하도 수 많은 쿼리가 나갈 수 있다.)
- 모든 컬럼을 묶어서 기본 키로 구성하기 때문에 컬럼에 null을 입력할 수 없고, 같은 값을 중복해서 저장할 수 없다.
- 따라서, 값 타입 컬렉션보다 새로운 엔티티를 만들어 `일대다 관계`에 `영속성 전이` + `고아 객체 제거` 기능을 추가해 컬렉션 처럼 사용하는 것이 좋다.

### 6.4 값 타입 컬렉션 대신 일대다 관계 매핑
- 일대다 단방향 매핑이기 때문에 외래 키가 다인 쪽에 있어 Address에 Update 쿼리가 나가는 것은 어쩔 수 없다.

```java
@Entity
public class AddressEntity {

    @Id @GeneratedValue
    private Long id;

    @Embedded Address address;

    public AddressEntity {
    }

    public AddressEntity(String city, String street, String zipcode) {
        this.address = new Address(city, street, zipcode);
    }
}

@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
    @JoinColumn(name = "MEMBER_ID")
    private List<AddressEntity> addressHistory = new ArrayList<>();
}
```

```java
Member member = new Member();

member.getAddressHistory().add(new AddressEntity("서울", "강남", "123-123"));
member.getAddressHistory().add(new AddressEntity("서울", "강북", "000-000"));

em.persist(member);
```

## 7. 엔티티 타입 vs 값 타입

### 7.1 엔티티 타입
- 식별자가 있다.
- 생명 주기가 있다.
- 공유할 수 있다.

### 7.2 값 타입
- 식별자가 없다.
- 스스로는 생명 주기가 없고 생명 주기를 엔티티에 의존한다.
- 공유하지 않는 것이 안전하다. (불변 객체로 만든다.)

## Reference
> 자바 ORM 표준 JPA 프로그래밍 [김영한]

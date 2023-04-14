# 고급 매핑

## 1. 상속 관계 매핑
상속관계 매핑: 객체의 상속 구조를 DB의 슈퍼타입 서브타입관계로 매핑

- RDB에는 상속이라는 개념이 없지만 `슈퍼타입 서브타입 관계`라는 모델링 기법이 있다. 
- 슈퍼타입 서브타입 논리 모델을 실제 물리 모델로 구현하는 방법으로 `조인 전략`, `단일 테이블 전략`, `구현 클래스마다 테이블 전략`이 있다.
- 기본으로 조인 전략을 선택하고 DB에 변경이 없는 간단한 엔티티에는 단일 테이블 전략을 선택해도 된다.

### 1.1 조인 전략
엔티티 각각을 테이블로 만들고 자식 테이블이 부모 테이블의 기본 키를 받아서 기본 키 + 외래 키로 사용하는 전략

- 객체는 타입으로 구분할 수 있지만 테이블은 타입이라는 개념이 없기 때문에 타입을 구분하는 DTYPE 컬럼을 사용한다.
- JPA 표준 명세는 구분 컬럼을 사용하도록 하지만 하이버네이트를 포함한 몇몇 구현체는 구분 컬럼 없이도 동작한다.

<img src="https://user-images.githubusercontent.com/108064146/231762524-0a1bc074-b8b8-487b-9590-105ae66a8776.png">

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {

    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;

    private String name;
    private int price;
}

@Entity
@DiscriminatorValue("A")
public class Album extends Item {

    private String artist;
}

@Entity
@PrimaryKeyJoinColumn(name = "MOVIE_ID")
public class Movie extends Item {

    private String director;
    private String actor;
}

@Entity
public class Book extends Item {

    private String author;
    private String isbn;
}
```

#### 조인 전략 애노테이션
- `@Inheritance(strategy = InheritanceType.JOINED)`: 매핑 전략 지정
- `@DiscriminatorColumn(name = "DTYPE")`: 부모 클래스의 구분 컬럼 지정 (생략 가능)
- `@DiscriminatorValue("A")`: 엔티티를 저장할 대 구분 컬럼에 입력할 값 지정
- `@PrimaryKeyJoinColumn(name = "MOVIE_ID")`: 자식 테이블의 기본키 칼럼명을 지정한 컬럼명으로 변경

#### 장점
- 테이블이 정규화된다.
- 외래 키 참조 무결성 제약조건을 활용할 수 있다.
- 저장공간을 효율적으로 사용한다.

#### 단점
- 조회할 때 조인을 많이 사용하기 때문에 성능이 저하될 수 있다.
- 조회 쿼리가 복잡하다.
- 데이터를 등록할 때 INSERT SQL을 두 번 실행한다.

### 1.2 단일 테이블 전략
하나의 테이블에서 여러 엔티티를 구분컬럼을 이용해 구분하는 전략

<img src="https://user-images.githubusercontent.com/108064146/231767603-09e50d0d-6c5f-41b2-aa78-b8ac93f54998.png">

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {

    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;

    private String name;
    private int price;
}

@Entity
public class Album extends Item {...}

@Entity
public class Movie extends Item {...}

@Entity
public class Book extends Item {...}
```

#### 장점
- 조인이 필요 없으므로 조회 성능이 빠르다.
- 조회 쿼리가 단순하다.

#### 단점
- 자식 엔티티가 매핑한 컬럼은 모두 null을 허용해야 한다.
- 단일 테이블에 모든 것을 저장하므로 테이블이 커질 수 있다.

### 1.3 구현 클래스마다 테이블 전략 (비추)
자식 엔티티마다 테이블을 만들어 필요한 컬럼들을 각각의 테이블에 모두 넣는 전략

<img src="https://user-images.githubusercontent.com/108064146/231769538-21fdd69a-cba2-41ab-bbed-0d821f1641e5.png">


```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Item {

    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;

    private String name;
    private int price;
}

@Entity
public class Album extends Item {...}

@Entity
public class Movie extends Item {...}

@Entity
public class Book extends Item {...}
```

#### 장점
- 서브 타입을 구분해서 처리할 때 효과적이다.
- not null 제약조건을 사용할 수 있다.

#### 단점
- 여러 자식 테이블을 함께 조회할 때 성능이 느리다. (UNION을 사용해야함.)
- 자식 테이블을 통합해서 쿼리하기 어렵다.

## 2. @MappedSuperclass
`@MappedSuperclass`로 지정한 부모 클래스는 테이블에 매핑하지 않고 자식 클래스에게 매핑 정보만 제공한다.
- `@MappedSuperclass`로 지정한 클래스는 엔티티가 아니므로 `em.find()`나 `JPQL`에서 사용할 수 없다.
- 주로 등록일, 수정일, 등록자, 수정자 같은 전체 엔티티에서 공통으로 적용하는 정보를 모을 때 사용

<img src="https://user-images.githubusercontent.com/108064146/231794926-860248a4-88c5-48dd-9904-50139c80958c.png">

```java
@MappedSuperclass
public abstract class BaseEntity {

    @Id @GeneratedValue
    private Long id;
    private String name;

    @ManyToOne
    private Address address;

    @ManyToOne
    private Team team;
}

@Entity
public class Member extends BaseEntity {
    private String email;
}

@Entity
public class Seller extends BaseEntity {
    private String shopName;
}
```

### 2.1 @AttributeOverride
부모로부터 물려받은 매핑 정보를 재정의
```java
@Entity
@AttributeOverride(name = "id", column = @Column(name = "MEMBER_ID"))
public class Member extends BaseEntity {...}

@Entity
@AttirbuteOverrides({
    @AttributeOverride(name = "id", column = @Column(name = "MEMBER_ID")),
    @AttributeOverride(name = "name", column = @Column(name = "MEMBER_NAME"))
})
public class Seller extends BaseEntity {...}
```

### 2.2 @AssociationOverride
부모로부터 물려받은 연관관계를 재정의

```java
@AssociationOverride(name = "address", joinColumns = @JoinColumn(name = "MEMBER_ADDRESS_ID"))
public class Member extends BaseEntity {...}

@AssociationOverrides({
    @AssociationOverride(name = "address", joinColumns = @JoinColumn(name = "SELLER_ADDRESS_ID")),
    @AssociationOverride(name = "team", joinColumns = @JoinColumn(name = "SELLER_TEAM_ID"))
})
public class Seller extends BaseEntity {...}
```

## 3. 복합키와 식별 관계 매핑
- 복합 키에는 `@GenerateValue`를 사용할 수 없다.
### 3.1 식별관계 vs 비식별 관계

- `식별 관계`: 외래 키를 기본 키로 사용 (실선)
- `비식별 관계`: 외래 키를 기본 키로 사용 X (점선)
- `필수적 비식별 관계`: 외래 키에 NULL 허용 X
- `선택적 비식별 관계`: 외래 키에 NULL 허용
- 기본으로 비식별 관계를 사용하고 꼭 필요한 곳에 식별 관계를 사용
- 선택적 비식별 관계보다는 필수적 비식별 관계를 사용하는 것이 좋다.
    + 선택적 비식별 관계는 NULL을 허용하므로 조인할 때 외부 조인을 사용
    + 필수적 비식별 관계는 NULL을 허용하지 않으므로 조인할 때 내부 조인만 사용해도 된다.
    
### 3.2 복합키 비식별 관계 - @IdClass

<img src = "https://user-images.githubusercontent.com/108064146/231806507-ddf5a6b2-add2-4229-a1b7-be4855a7a58c.jpeg">

#### 식별자 클래스 조건
- 식별자 클래스의 속성명과 엔티티에서 사용하는 식별자의 속성명이 같아야한다. (ParentId.`id1` 명 == Parent.`id1` 명)
- `Serializable` 인터페이스를 구현
- `equals`, `hashCode`를 구현
- 기본 생성자
- 식별자 클래스는 `public`이어야한다.

```java
public class ParentId implements Serializable {

    private Long id1;
    private Long id2;

    public ParentId() {
    }

    public ParentId(Long id1, Long id2) {
        this.id1 = id1;
        this.id2 = id2;
    }

    @Override
    public boolean equals(Object o) {...}

    @Override
    public int hashCode() {...}
}

@Entity
@IdClass(ParentId.class)
public class Parent {

    @Id 
    @Column(name = "PARENT_ID1")
    private Long id1;

    @Id
    @Column(name = "PARENT_ID2")
    private Long id2;

    private String name;
}

@Entity
public class Child {

    @Id
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumns({
        @JoinColumn(name = "PARENT_ID1"),
        @JoinColumn(name = "PARENT_ID2")
    })
    private Parent parent;
}

// 저장
Parent parent = new Parent();
parent.setId1(1L);
parent.setId2(2L);
parent.setName("parentName");
em.persist(parent)

// 조회
ParentId parentId = new ParentId(1L, 2L);
Parent parent = em.find(Parent.Class, parentId);
```

### 3.3 복합키 비식별 관계 - @EmbeddedId

#### 식별자 클래스 조건
- `@Embeddable` 어노테이션을 붙여준다.
- `Serializable` 인터페이스 구현
- `equals`, `hashCode` 구현
- 기본 생성자
- 식별자 클래스는 `public`이어야한다.
```java
@Embeddable
public class ParentId implements Serializable {

    @Column(name = "PARENT_ID1")
    private Long id1;

    @Column(name = "PARENT_ID2")
    private Long id2;

    public ParentId() {
    }

    public ParentId(Long id1, Long id2) {
        this.id1 = id1;
        this.id2 = id2;
    }

    @Override
    public boolean equals(Object o) {...}

    @Override
    public int hashCode() {...}
}

@Entity
public class Parent {

    @EmbeddedId
    private ParentId id;

    private String name;
}

@Entity
public class Child {

    @Id
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumns({
        @JoinColumn(name = "PARENT_ID1"),
        @JoinColumn(name = "PARENT_ID2")
    })
    private Parent parent;
}

// 저장
Parent parent = new Parent();
ParentId parentId = new ParentId(1L, 2L);
parent.setId(parentId);
parent.setName("parentName");
em.persist(parent);

// 조회
ParentId parentId = new ParentId(1L, 2L);
Parent parent = em.find(Parent.class, parentId);
```

## Reference
> 자바 ORM 표준 JPA 프로그래밍 [김영한]

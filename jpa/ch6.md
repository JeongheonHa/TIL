# 다양한 연관관계 매핑

## 1. 다대일

### 1.1 다대일 단방향
연관관계 주인이 다 쪽에 있다.
```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}


@Entity
public class Team {

    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    private String name;
}
```

### 1.2 다대일 양방향

```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    private void changeTeam(Team team) {
        this.team = team;

        // 무한 루프에 빠지지 않도록
        if (!team.getMembers().contains(this)) {
            team.getMembers().add(this);
        }
    }
}

@Entity
public class Team {

    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    private String name;

    @OneToMany(mappedBy = "team")
    List<Member> members = new ArrayList<>();

    public void addMember(Member member) {
        this.members.add(member);

        // 무한 루프에 빠지지 않도록
        if (member.getTeam() != this) {
            member.setTeam(this);
        }
    }
}
```

## 2. 일대다 (비추)

### 2.1 일대다 단방향

<img src="https://user-images.githubusercontent.com/108064146/231746554-9d1ffb60-51c3-4d9c-8572-8368767e2e3a.png">

```java
@Entity
public class Team {

    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    private String name;

    @OneToMany
    @JoinColumn(name = "TEAM_ID")
    private List<Member> members = new ArrayList<>();
}

@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;
}
```

#### 일대다 단방향 매핑의 단점
외래 키는 다인 쪽에 있지만 연관관계 주인은 1인 쪽에 있기 때문에 team에 member를 추가할 경우 update 쿼리가 team이 아닌 member 테이블을 대상으로 나간다.

### 2.2 일대다 양방향
일대다 양방향 매핑은 존재하지 않기 때문에 다대일 양방향 매핑을 사용해야한다. 하지만 아래와 같디 꼼수를 이용하면 양방향 매핑인 것처럼 만들 수 있다.

<img src="https://user-images.githubusercontent.com/108064146/231746664-3c0246f0-424c-4e5a-88f6-edd9c4d25225.png">

```java
@Entity
public class Team {

    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    private String name;

    @OneToMany
    @JoinColumn(name = "TEAM_ID")
    List<Member> members = new ArrayList<>();
}

@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID", insertable = false, updatable = false)
    private team team;
}
```

## 3. 일대일
일대일 관계는 주 테이블과 대상 테이블 중 외래 키를 누가 가질지 선택해야한다.

### 3.1 주 테이블에 외래 키
주 객체가 대상 객체를 참조하는 것처럼 객체지향적으로 설계 가능

- 장점: 주 테이블만 조회해도 대상 테이블에 데이터가 있는지 확인 가능
- 단점: 값이 없으면 외래 키에 null 허용

#### 주 테이블에 일대일 단방향

<img src="https://user-images.githubusercontent.com/108064146/231746935-394f4f3c-72f6-40ae-8fc5-7ae8e0017de2.png">

```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;
}

@Entity
public class Locker {

    @Id @GeneratedValue
    @Column(name = "LOCKER_ID")
    private Long id;

    private String name;
}
```

#### 주 테이블에 일대일 양방향

<img src="https://user-images.githubusercontent.com/108064146/231748067-73443122-1928-4951-afa8-97162648c16d.png">

```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;
}

@Entity
public class Locker {

    @Id @GeneratedValue
    @Column(name = "LOCKER_ID")
    private Long id;

    private String name;

    @OneToOne(mappedBy = "locker")
    private Member member;
}
```

### 3.2 대상 테이블에 외래 키

- 장점: 주 테이블과 대상 테이블을 일대일에서 일대다 관계로 변경할 때 테이블 구조 유지 (unique 옵션만 변경하면 된다.)
- 단점: 프록시 기능의 할계로 지연 로딩으로 설정해도 항상 즉시 로딩됨

#### 대상 테이블에 일대일 단방향
JPA에서는 대상 테이블에 외래키가 있는 단방향 관계는 지원하지 않고 이렇게 매핑할 수 도 없다.

<img src="https://user-images.githubusercontent.com/108064146/231748728-2283aeda-6a83-41e4-bc96-df5d34450edd.png">

#### 대상 테이블에 일대일 양방향
연관관계 주인을 대상 엔티티가 갖도록 한다.

<img src="https://user-images.githubusercontent.com/108064146/231749953-ed57a6bb-ad2c-4a03-812d-8a34514703bc.png">

```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    @OneToOne(mappedBy = "member")
    private Locker locker;
}

@Entity
public class Locker {

    @Id @GeneratedValue
    @Column(name = "LOCKER_ID")
    private Long id;

    private String name;

    @OneToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;
}
```

## 4. 다대다
- `DB 테이블`: 테이블에서는 다대다 관계를 일대다, 다대일 관계로 풀어서 테이블을 설계한다.
- `JPA`: `@JoinTable`을 통해 다대다 관계를 표현할 수 있지만 일대다, 다대일 관계로 풀어서 설계하는 것이 좋다.

### 4.1 다대다 테이블 설계
다대다 테이블을 설계할 때는 외래 키들을 기본 키로 사용하는 `식별 관계`와 기본 키가 따로 있고 외래 키들을 속성으로 사용하는 `비식별 관계` 중 하나를 골라 관계 테이블을 설계한다.

- 요즘은 관계 테이블의 식별자 값을 따로 지정하고 외래 키들을 속성값으로 사용한다.

### 4.1 다대다 단방향 + 양방향

<img  src="https://user-images.githubusercontent.com/108064146/231755791-f11a4ca0-834c-42a9-99c7-471b7be378e5.png">

```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();
}

@Entity
public class Order {

    @Id @GeneratedValue
    @Column(name = "ORDER_ID")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;

    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Product product;

    private int orderAmount;
}

@Entity
public class Product {

    @Id @GeneratedValue
    @Column(name = "PRODUCT_ID")
    private Long id;

    private String name;
}
```

## Reference
> 자바 ORM 표준 JPA 프로그래밍 [김영한]

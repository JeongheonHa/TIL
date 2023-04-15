# 프록시와 영속성 전이

## 1. 프록시 기초
- `상황`: 멤버 엔티티만 조회하고 싶은 경우 멤버 엔티티와 연관된 팀 엔티티까지 가져오는 것은 효율적이지 않다.
- `해결`: JPA는 프록시 객체를 반환함으로써 엔티티가 실제 사용될 때까지 데이터베이스 조회를 지연하는 `지연 로딩`을 지원한다.
- JPA는 지연 로딩의 구현 방법을 JPA 구현체에 위임

```java
// 프록시 객체를 직접 반환 (DB 조회 X, 실제 엔티티 생성 X)
Member member = em.getReference(Member.class, "member1");

member.getName(); // 1. 메서드 호출
```

```java
// 프록시 클래스 예상 코드
class MemberProxy extends Member {

    Member target = null; // 실제 엔티티 참조

    public String getName() {

        if (target == null) {

            // 2. 초기화 요청
            // 3. DB 조회
            // 4. 실제 엔티티 생성 및 참조 보관
        }

        // 5. target.getName();
        return target.getName();
    }
}
```
### 1.1 프록시의 특징
- 프록시 클래스는 실제 클래스를 상속 받아서 만들어진다.

    + 사용자는 해당 객체가 프록시인지 실제 객체인지 구분하지 않고 사용 가능
    + 타입 체크시 주의

- 프록시 객체는 실제 객체에 대한 참조(target)를 보관한다.

- 프록시 객체의 메서드를 호출하면 실제 객체의 해당 메서드를 호출한다.

- 프록시 객체는 처음 사용할 때 한번만 초기화된다.

- 프록시 객체를 초기화한다고 실제 엔티티로 바뀌는 것이 아니다.

- 영속성 컨텍스트에 찾는 엔티티가 있는 경우 em.getReference()를 호출해도 프록시가 아닌 실제 엔티티가 반환된다.

    + 반대로 영속성 컨텍스트에 프록시 객체가 있는 경우 em.find()를 호출해도 프록시 객체가 반환된다.
    + 즉, JPA는 같은 트랜잭션 내에서 같은 객체를 호출하면 동일성을 보장한다.

- 초기화는 영속성 컨텍스트의 도움을 받아야 가능하기 때문에 영속 상태가 아닌 프록시를 초기화하면 예외를 발생시킨다.
    + hibernate는 org.hibernate.LazyInitializationExcepton 예외 발생

### 1.2 프록시 객체 초기화

<img src="https://user-images.githubusercontent.com/108064146/231896846-e2d5cd99-1e45-4db1-a6cb-01e6578c3228.png">

1. 프록시 객체에 member.getName()을 호출

2. 실제 엔티티가 생성되어 있지 않으면 영속성 컨텍스트에 실제 엔티티 생성을 요청 (if. target == null)

3. 영속성 컨텍스트는 DB를 조회해서 실제 엔티티 객체를 생성

4. 프록시 객체는 생성된 엔티티 객체의 참조를 target 멤버변수에 보관

5. 프록시 객체는 실제 엔티티 객체의 getName()을 호출해서 결과를 반환

### 1.3 프록시와 식별자
엔티티를 프록시로 조회할 때 식별자(PK) 값을 파라미터로 전달하는데 프록시 객체는 이 식별자 값을 보관한다.

- `프로퍼티 접근 방식의 경우`: getId()메서드는 식별자를 매핑하는 일 밖에 하지 않는다. 따라서, `초기화 X`
- `필드 접근 방식의 경우`: 필드로 식별자를 매핑하는 것이기 때문에 getId()메서드는 단순 메서드일 뿐 내부에서 어떤 일하는지 알지 못한다. 따라서, `초기화 O`
- `연관관계를 설정하는 경우`: 연관관계를 설정할 때는 필드 접근 방식이여도 `초기화 X`

### 1.4 프록시 확인
프록시 인스턴스의 초기화 여부를 확인 (true = 초기화 O, false = 초기화 X)
```java
boolean isLoad = emf.getPersistenceUnitUtil().isLoaded(entity);
```

### 1.5 프록시 강제 초기화
- `JPA 표준`: 프록시 강제 초기화 메서드가 없으므로 프록시 객체 메서드를 직접 호출 (ex. member.getName())
- `hibernate`: `Hibernate.initialize(proxy객체)`를 호출하면 프록시를 강제 초기화 할 수 있다.

## 2. 즉시 로딩과 지연 로딩

### 2.1 즉시 로딩 (비추)
엔티티를 조회할 때 연관된 엔티티도 함께 조회

- 대부분의 JPA 구현체들은 즉시 로딩을 최적화하기 위해 가능하면 조인 쿼리를 사용한다.

```java
@ManyToOne(fetch = FetchType.EAGER)
```

### 2.2 외부 조인과 내부 조인
외부 조인보다 내부 조인이 성능과 최적화에서 더 유리하다.
- JPA는 `선택`적 관계면 `외부 조인`을 사용
- JPA는 `필수`적 관계면 `내부 조인`을 사용
- `NOT NULL` 옵션을 추가함으로써 선택 관계를 필수 관계로 만들면 내부 조인 사용 가능

### 2.3 지연 로딩
연관된 엔티티를 실제 사용할 때 조회

- 실무에서는 모두 지연 로딩을 사용해야한다.

```java
@ManyToOne(fetch = FetchType.LAZY)
```

```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    private Integer age;

    @ManyToOne(fetch = FetchType.EAGER)
    private Team team;

    @OneToMany(mappedBy = "member", fetch = FetchType.LAZY)
    private List<Order> orders;
}
```

### 2.4 프록시와 컬렉션 래퍼
`컬렉션 래퍼`: 하이버네이트가 엔티티를 영속 상태로 만들 때 엔티티에 컬렉션이 있으면 컬렉션을 추적하고 관리할 목적으로 원본 컬렉션을 하이버네이트가 제공하는 내장 컬렉션으로 변경하는데 이것을 컬렉션 래퍼라고 한다. 즉, 컬렉션 프록시 객체라고 생각하면 된다.

### 2.5 JPA 기본 페치 전략
연관된 엔티티가 하나면 즉시 로딩, 컬렉션이면 지연 로딩을 사용 (컬렉션을 즉시 로딩하면 너무 많은 데이터를 로딩할 수 있다.)
- `@ManyToOne`, `@OneToOne`: 즉시 로딩
- `@OneToMany`, `@ManyToMany`: 지연 로딩
- 모든 연관관계에 지연 로딩을 사용하는 것이 좋다.

### 3.6 컬렉션에 즉시 로딩 사용시 주의점
- 컬렉션을 하나 이상 즉시 로딩하는 것은 권장하지 않는다.

    + 컬렉션 * 컬렉션 * 컬렉션으로 테이블의 카디널리티가 배로 증가한다.

- 컬렉션 즉시 로딩은 항상 외부 조인을 사용한다.

    + 내부 조인시 해당 엔티티의 데이터가 포함되지 않을 수 있다. (ex. Member(N) : Team(1)에서 어떤 팀에는 멤버가 없을 수도 있다.)

- `@ManyToOne`, `@OneToOne`
    + (optional = false): 내부 조인
    + (optional = true): 외부 조인
- `@OneToMany`, `@ManyToMany`
    + (optional = false): 외부 조인
    + (optional = true): 외부 조인

## 4. 영속성 전이: CASCADE
특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶을때 사용
- JPA는 엔티티를 영속 상태로 만들 때 연관된 모든 엔티티가 영속 상태여야 하지만 영속성 전이를 사용하면 부모만 영속 상태로 만들면 연관된 자식까지 한 번에 영속 상태로 만들 수 있다.
- 특정 엔티티가 개인 소유하는 엔티티에만 사용
- 연관된 두 엔티티가 생명 주기가 같을 때 사용

### 4.1 영속성 전이 기능 사용
```java
@Entity
public class Parent {

    @Id @GeneratedValue
    private Long id;

    // Parent를 persist할 때 cascade 옵션이 붙은 참조에 있는 모든 엔티티를 persist한다.
    @OneToMany(mappedBy = "parent", cascade = {CascadeType.PERSIST, CascadeType.REMOVE})
    private List<Child> children = new ArrayList<>();
}

@Entity
public class Child {

    @Id @GeneratedValue
    private Long id;

    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    private Parent parent;
}
```

### 4.2 엔티티 저장
```java
Child child1 = new Child();
Child child2 = new Child();
Parent parent = new Parent();

child1.setParent(parent); // 연관관계 추가
child2.setParent(parent); // 연관관계 추가

parent.getChildren().add(child1);
parent.getChildren().add(child2);

em.persist(parent); // 부모만 영속 상태 -> 관련된 자식 모두 영속 상태로 만듬
```

### 4.3 엔티티 제거
```java
Parent findParent = em.find(Parent.class, 1L);
em.remove(findParent);  // 부모만 제거하면 관련된 자식 모두 제거 (외래 키 무결성 제약 조건 만족)
```

### 4.4 CASCADE 종류
ALL, PERSIST, REMOVE 정도를 자주 사용한다.
```java
public enum CascadeType {

    ALL,     // 모두 적용
    PERSIST, // 영속
    REMOVE,  // 제거
    MERGE,   // 병합
    REFRESH, // REFRESH
    DETACH   // DETACH

}
```

## 5. 고아 객체 제거
부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동을 제거
- `flush()`호출시 Delete 쿼리 실행
- 특정 엔티티가 개인 소유하는 엔티티에만 사용
- 연관된 두 엔티티가 생명 주기가 같을 때 사용
- `@OneToOne`, `@OneToMany`에만 사용
- 부모를 제거하면 컬렉션이 제거되므로 자식들도 모두 같이 제거된다. (like. CascadeType.REMOVE)
```java
@Entity
public class Parent {

    @Id @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "parent", orphanRemoval = true)
    private List<Child> children = new ArrayList<Child>();

    //getter, setter
}

Parent parent1 = em.find(Parent.class, 1L);
parent1.getChildren().remove(0); // 참조 제거

// 모든 엔티티 제거
parent1.getChildren().clear();
```

## 6. 영속성 전이 + 고아객체, 생명주기
`CascadeType.All` + `orphanRemoval = true`를 동시에 활성화하면 부모 엔티티로 자식의 생명주기를 관리할 수 있다. (DAO나 Repository 없이 부모 엔티티가 자식 엔티티를 관리)

```java
@Entity
public class Parent {

    @Id @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Child> children = new ArrayList<Child>();

    public void addChild(Child child) {
        children.add(child);
        child.setParent(this);
    }

    //getter, setter
}

Parent parent = em.find(Parent.class, parentId);
Child child1 = new Child();

// 자식 저장 (cacade)
parent.addChild(child1);
em.persist(parent);

// 자식 제거 (orphanRemoval)
parent.getChildren().remove(child1);
em.flush();
```

## Reference
> 자바 ORM 표준 JPA 프로그래밍 [김영한]

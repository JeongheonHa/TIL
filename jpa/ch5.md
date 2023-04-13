# 연관관계 매핑

## 1. 단방향 연관관계

### 1.1 객체 연관관계
- 객체는 참조로 연관관계를 맺는다.
- 단방향 관계

### 1.2 테이블 연관관계
- 테이블은 외래 키로 연관관계를 맺는다.
- 양방향 관계

## 2. 객체 관계 매핑

<img src="https://user-images.githubusercontent.com/108064146/231484087-9534a5f2-a4de-4be4-8300-1564be789412.png">

```java
@Entity
public class Member {

    @Id
    @Column(name = "MEMBER_ID")
    private String id;

    private String username;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    public void setTeam(Team team) {
        this.team = team;
    }
}

@Entity
public class Team {
    
    @Id
    @Column(name = "TEAM_ID")
    private String id;

    private String name;
}
```

### 2.1 @JoinColumn
외래 키를 매핑할 때 사용
- 생략시 `필드명 + _ + 참조하는 테이블의 컬럼명`인 외래 키를 사용한다.
#### @JoinColumn 속성
- `@JoinColumn.name`: 매핑할 외래 키 이름 (기본값: 필드명 + _ + 참조하는 테이블의 기본 키 컬럼명)
- `@JoinColumn.referencedColumnName`: 외래 키가 참조하는 대상 테이블의 컬럼명 (참조하는 테이블의 기본키 컬럼명)
- `@JoinColumn.foreignKey`: 외래키 제약 조건 지정
- 이하 @Column의 속성


### 2.2 @ManyToOne
다대일 관계라는 매핑 정보

#### @ManyToOne 주요 속성
- `@ManyToOne.optional`: false로 설정하면 연관된 엔티티가 항상 있어야 한다. (기본값: true)
- `@ManyToOne.fetch`: 글로벌 페치 전략을 설정 (기본값: @ManyToOne=FetchType.EAGER, @OneToMany=FetchType.LAZY)
- `@ManyToOne.cascade`: 영속성 전이 기능을 사용

## 3. 등록, 수정, 삭제, 조회

### 3.1 저장
JPA에서 엔티티를 저장할 때 연관된 모든 엔티티는 영속 상태여야 한다.

```java
Team team = new Team("team1", "레드팀");
em.persist(team);

Member member = new Member("member1", "이수근")
member.setTeam(team);
em.persist(member)
```

### 3.2 조회

#### 객체 그래프 탐색
```java
Member member = em.find(Member.class, "member1");
Team team = member.getTeam();
```

#### 객체 지향 쿼리 이용
```java
String jpql = "select m from Member m join m.team t where " + "t.name=:teamName";

List<Member> resultList = em.createQuery(jpql, Member.class)
    .setParameter("teamName", "레드팀")
    .getResultList();
```

### 3.3 수정
```java
Team team2 = new Team("team2", "블루팀");
em.persist(team2);

Member member = em.find(Member.class, "member1");
member.setTeam(team2);
```

### 3.4 제거
외래 키 무결성 제약 조건에 의해 외래 키는 없는 값을 참조할 수 없기 때문에 team을 제거하기 전에 연관 관계를 제거해야한다.
```java
Member member = em.find(Member.class, "member1");
Team team = member.getTeam();

member.setTeam(null); // 연관관계 제거
em.remove(team); // 엔티티 제거
```

## 4. 양방향 연관관계
- 테이블은 외래 키 하나로 양방향 연관관계를 맺는다.
- 객체는 참조를 2개를 사용해 양방향 연관관계를 맺는다.

<img src="https://user-images.githubusercontent.com/108064146/231496696-22d55dfb-f5b9-4b00-afe9-9ea404b09c59.png">

```java
@Entity
public class Team {

    @Id
    @Column(name = "TEAM_ID")
    private String id;

    private String name;

    // 추가한 부분
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();
}
```

### 4.1 일대다 컬렉션 조회
```java
Team team = em.find(Member.class, "레드팀");
List<Member> members = team.getMembers(); // 객체 그래프 탐색
```

### 4.2 연관관계 주인
연관관계의 주인만이 데이터베이스 연관관계와 매핑되고 외래 키를 관리할 수 있고 반대편은 읽기만 할 수 있다.
- 외래 키가 있는 쪽을 연관관계 주인으로 정한다.

```java
@OneToMany(mappedBy = "team") // team이라는 참조 값을 연관관계의 주인으로 지정
```

### 4.3 등록, 수정, 삭제, 조회

#### 저장
연관관계 주인이 아닌 참조에는 등록, 수정, 삭제가 불가능하기 때문에 JPA는 `team1.getMembers().add(member1)`을 DB에 반영하지 않는다.
- 객체 관점에서 봤을 때 반대쪽 참조에도 값을 넣어주는 것이 자연스럽기 때문에 추가하는 것이 좋다.
- 한 트랜잭션 내에서 member를 저장하고 team의 members에서 member를 찾을 때 flush()를 하지 않았기 때문에 DB에 member가 저장되지 않아 찾을 수 없다. 하지만 해당 코드를 통해 영속성 컨텍스트에 team 엔티티의 members에 member를 저장함으로써 데이터 정합성을 지킬 수 있다.
```java
Team team1 = new Team("team1", "레드팀");
em.persist(team1);

Member member1 = new Member("member1", "이수근");

// 양방향 연관관계 매핑
member1.setTeam(team1);
team1.getMembers().add(member1);
em.persist(member1);

Member member2 = new Member("member2", "김병만");

// 양방향 연관관계 매핑
member2.setTeam(team1);
team1.getMembers().add(member2);
em.persist(member2);
```

#### 연관관계 편의 메서드
- 양방향 관계에서는 연관관계 편의 메서드를 만들어 연관관계를 매핑하는 것이 안전하다.
- 양방향 관계에서는 이전의 연관관계가 있는 경우 반대쪽 참조의 연관관계를 제거해야한다.
```java
@Entity
public class Member {

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    public void changeTeam(Team team) {

        if (this.team != null) {
            this.team.getMembers().remove(this);
        }

        this.team = team;
        team.getMembers().add(this);
    }
}
```

### 4.4 양방향 매핑시 주의사항

#### toString() 사용시 주의
Object 클래스를 상속받는 클래스들은 참조 변수만을 호출하면 .toString()을 자동으로 생성해서 호출하기 때문에 양방향 연관관계의 두 참조를 toString()하면 무한 루프에 빠지게 된다.
- member.toString() 호출 -> team.toString() 호출 -> members.toString() 호출 -> member.toString() 호출 ... 무한 반복

```java
@Entity
public class Member {

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    @Override
    public String toString() {
        return "Member{" + 
                ", team=" + team.toString() + 
                '}';
    }
}

@Entity
public class Team {
    
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();

    @Override
    public String toString() {
        return "Team{" +
                ", members=" + members.toString() +
                '}';
    }
}
```

#### JSON 변환시 주의
컨트롤러에서 엔티티를 그대로 JSON으로 변환할 경우 양방향 연관관계의 두 엔티티를 무한으로 JSON 형식으로 변환한다. 

- Member.team JSON 변환 -> Team.members JSON 변환 -> member JSON 변환 -> Member.team JSON 변환 ... 무한 반복
- 엔티티의 데이터를 DTO에 저장해서 전달하는 것이 안전하다.

## Reference
> 자바 ORM 표준 JPA 프로그래밍 [김영한]

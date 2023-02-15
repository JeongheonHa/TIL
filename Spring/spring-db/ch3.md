# Transaction

## 1. 트랜잭션의 특성 (ACID)
- `원자성`(Atomicity)
- `일관성`(Consistency)
- `격리성`(Isolation)
- `지속성`(Durability)

### 1.1 트랜잭션 격리 수준 (Isolation Level)
단계가 높아질 수 록 성능이 낮아진다.

- READ UNCIMMITED(커밋되지 않은 읽기, low)
- `READ COMMITED`(커밋된 읽기, 보통 이걸 많이 사용)
- `REPEATABLE READ`(반복 가능한 읽기)
- SERIALIZABLE(직렬화 기능, high)

## 2. DB 연결 구조
<img src = "https://user-images.githubusercontent.com/108064146/218988647-8d8b2dd7-7991-4b8e-8cdb-63c51139730a.png">

- DB 서버에 연결을 요청하고 커넥션은 맺는다.
- DB 서버 내부에 세션을 만들고 해당 커넥션을 통한 모든 요청은 해당 세션을 통해 실행된다.
- 세션은 트랜잭션을 시작하고, 커밋 또는 롤백을 통해 트랜잭션을 종료한다.
- 사용자가 커넥션을 닫거나, DBA가 세션을 강제로 종료하면 세션은 종료된다.
- 커넥션 풀이 10개의 커넥션을 생성하면, 세션도 10개 생성된다.

## 3. commit & rollback
- commit이나 rollback을 호출하기 전까지는 임시로 데이터를 저장한다.
- `commit`: DB에 변경 데이터를 저장한다.
- `rollback`: transaction 수행 이전으로 되돌린다. 
- 해당 트랜잭션을 시작한 세션에게만 변경 데이터가 보이고 다른 세션에게는 변경 데이터가 보이지 않는다.

## 4. 자동 커밋, 수동 커밋
- `자동 커밋`(기본 값): 각각의 쿼리 실행 직후에 자동으로 커밋을 호출한다. (트랜잭션 X)
- `수동 커밋`: commit과 rollback을 수동으로 호출한다. (트랜잭션 시작)

```sql
set autocommit false; // 수동 커밋 모드 설정
insert into member(member_id, money) values('data1', 10000);
insert into member(member_id, money) values('data2', 10000);
commit; // 수동 커밋
```

## 5. DB Lock
DB Lock: 한 세션이 트랜잭션을 시작하고 데이터를 수정하는 동안 커밋이나 롤백 전까지 다른 세션에서 해당 데이터를 수정할 수 없게 막는다.

<img src = "https://user-images.githubusercontent.com/108064146/218994771-432d45c7-80c2-44c1-91d8-3416e30dc987.png">

- 조금이라도 빨리 요청한 세션이 해당 low의 lock의 우선권을 갖는다.
- 락이 없으면 락이 돌아올 때 까지 대기한다.
- 락 대기 시간을 넘어가면 락 타임아웃 오류가 발생한다.
- 커밋으로 트랜잭션이 종료되면 락을 반납한다.

```sql
// 락 대기 시간 설정 (60초)
SET LOCK_TIMEOUT 60000;
set autocommit false;
update member set money=1000 where member_id = 'memberA';
commit;
```

### 5.1 DB LOCK - 조회(select)
- 일반적인 조회는 락을 회득하지 않고 데이터를 조회할 수 있다.

#### 데이터를 조회할 때도 락을 얻고 싶은 경우
- `select for update`를 사용
- 해당 세션이 조회(select)를 수행할 때도 락을 얻어 다른 세션에서 해당 데이터에 접근할 수 없다.

```sql
set autocommit false;
select * from member where member_id = 'memberA' for update;
commit; // 락 반납
```

## 6. 트랜잭션 적용
<img src = "https://user-images.githubusercontent.com/108064146/218998958-1241389a-fbce-463d-bbb6-5bdd07d02fe4.png">

- 트랜잭션은 비즈니스 로직이 있는 Service 계층에서 시작해야 한다.
    + 비즈니스 로직이 잘못되면 rollback을 해야하기 때문
- 트랜잭션을 시작하려면 커넥션이 필요하기 때문에 Service 계층에서 커넥션을 만들고 종료해야한다.
- 같은 세션을 사용하기 위해 트랜잭션을 사용하는 동안 같은 커넥션을 유지해야한다.

### 6.1 RepositoryV2
커넥션을 Service계층에서 파라미터로 받아서 사용한다.
```java
@Slf4j
public class MemberRepositoryV2 {

    private final DataSource dataSource;

    public MemberRepositoryV2(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public Member findById(Connection con, String memberId) throws SQLException {
        String sql = "select * from Member where member_id = ?";

        PreparedStatement pstmt = null;
        ResultSet rs = null;

        try {
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, memberId);
            rs = pstmt.executeQuery();

            if (rs.next()) {
                Member member = new Member();
                member.setMemberId(rs.getString("member_id"));
                member.setMoney(rs.getInt("money"));
                return member;
            } else {
                throw new NoSuchElementException("member not found memberId=" + memberId);
            }
        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            //connection은 여기서 닫으면 안된다.
            JdbcUtils.closeStatement(pstmt);
            JdbcUtils.closeResultSet(rs);
        }
    }

    public void update(Connection con, String memberId, int money) throws SQLException {
        String sql = "update member set money = ? where member_id = ?";

        PreparedStatement pstmt = null;
        try {
            pstmt = con.prepareStatement(sql);
            pstmt.setInt(1, money);
            pstmt.setString(2, memberId);
            pstmt.executeUpdate();
        } catch (SQLException e) {
            log.info("db error", e);
            throw e;
        } finally {
            //connection은 여기서 닫으면 안된다.
            JdbcUtils.closeStatement(pstmt);
        }
    }
}
```

### 6.2 MemberServiceV2
여기서 세션을 만들어 트랜잭션을 시작/종료 시키고 인자로 커넥션을 넘겨준다.
- `release(con)`: 커넥션 풀을 사용하면 `con.close()`를 호출했을 때 커넥션이 종료되는 것이 아닌 풀에 반납된다. 이 때 기본 값인 자동 커밋 상태로 변경하는 것이 안전하다.
```java
@Slf4j
@RequiredArgsConstructor
public class MemberServiceV2 {

    private final DataSource dataSource;
    private final MemberRepositoryV2 memberRepository;

    
    public void accountTransfer(String fromId, String toId, int money) throws SQLException {
        
        Connection con = dataSource.getConnection();
        try {
            con.setAutoCommit(false); //트랜잭션 시작
            bizLogic(con, fromId, toId, money);
            con.commit(); // 트랜잭션 종료
        } catch (Exception e) {
            con.rollback();
            throw new IllegalStateException(e);
        } finally {
            release(con);
        }
        //커밋 or 롤백
    }

    private static void release(Connection con) {
        if (con != null) {
            try {
                con.setAutoCommit(true); // 다시 오토 커밋 상태로 만들어준다.
                con.close(); // 커넥션 풀에 커넥션을 다시 반납
            } catch (Exception e) {
                log.info("error", e);
            }
        }
    }

    private void bizLogic(Connection con, String fromId, String toId, int money) throws SQLException {
        Member fromMember = memberRepository.findById(con, fromId);
        Member toMember = memberRepository.findById(con, toId);

        memberRepository.update(con, fromId, fromMember.getMoney() - money);
        validation(toMember);
        memberRepository.update(con, toId, toMember.getMoney() + money);
    }

    private static void validation(Member toMember) {
        if (toMember.getMemberId().equals("ex")) {
            throw new IllegalStateException("이체 중 예외 발생");
        }
    }
}
```

## 7. Test

```java
class MemberServiceV2Test {

    public static final String MEMBER_A = "memberA";
    public static final String MEMBER_B = "memberB";
    public static final String MEMBER_EX = "ex";

    private MemberRepositoryV2 memberRepository;
    private MemberServiceV2 memberService;

    @BeforeEach
    void before() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);
        memberRepository = new MemberRepositoryV2(dataSource);
        memberService = new MemberServiceV2(dataSource, memberRepository);
    }

    @AfterEach
    void after() throws SQLException {
        memberRepository.delete(MEMBER_A);
        memberRepository.delete(MEMBER_B);
        memberRepository.delete(MEMBER_EX);
    }

    @Test
    @DisplayName("이체 중 오류 발생")
    void accountTransferEx() throws SQLException {
        //given
        Member memberA = new Member(MEMBER_A, 10000);
        Member memberEx = new Member(MEMBER_EX, 10000);
        memberRepository.save(memberA);
        memberRepository.save(memberEx);

        //when
        assertThatThrownBy(() -> memberService.accountTransfer(memberA.getMemberId(), memberEx.getMemberId(), 3000))
                .isInstanceOf(IllegalStateException.class);

        //then
        Member findMemberA = memberRepository.findById(memberA.getMemberId());
        Member findMemberEx = memberRepository.findById(memberEx.getMemberId());
        //비즈니스 로직 실행 중 오류가 발생하면 rollback
        assertThat(findMemberA.getMoney()).isEqualTo(10000);
        assertThat(findMemberEx.getMoney()).isEqualTo(10000);
    }
}
```

## Reference
> 스프링 DB 1편 - 데이터 접근 핵심 원리 [김영한]

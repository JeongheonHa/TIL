# 예외 처리, 반복

## 1. 예외와 인터페이스

### 1.1 체크 예외와 인터페이스
구현체에서 체크 예외를 사용하려면 인터페이스에도 해당 체크 예외가 선언되어 있어야한다.
- 인터페이스의 목적은 구현체에 의존하지 않기 위해서인데 체크 예외를 사용하면 인터페이스가 구현체의 기술을 의존하게 된다.

### 1.2 런타임 예외와 인터페이스
런타임 예외는 인터페이스에 따로 예외를 선언하지 않아도 된다.

```java
//checked Exception 
public interface MemberRepositoryEx {
    Member save(Member member) throws SQLException;
    Member findById(String memberId) throws SQLException;
    void update(String memberId, int money) throws SQLException;
    void delete(String memberId) throws SQLException;
}
//unchecked Exception
public interface MemberRepository {
    Member save(Member member);
    Member findById(String memberId);
    void update(String memberId, int money);
    void delete(String memberId);
}
```

## 2. 리포지토리 인터페이스, 런타임 예외 적용
```java
//1. implement MemberRepository 
@Slf4j
public class MemberRepositoryV4_1 implements MemberRepository {

    private final DataSource dataSource;

    public MemberRepositoryV4_1(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    //2. throws SQLException 생략
    public Member save(Member member) throws SQLException {...}
    @Override
    public Member save(Member member) {
        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, member.getMemberId());
            pstmt.setInt(2, member.getMoney());
            pstmt.executeUpdate();
            return member;
        } catch(SQLException e) {
            //3. 체크 예외 -> 런타임 예외로 던지기
            throw new MyDbException(e);
        } finally {
            close(con, pstmt, null);
        }
    }

    //...
}
```

## 3. Service 계층 예외 누수 문제 해결

```java
@Slf4j
@RequiredArgsConstructor
public class MemberServiceV4 {
    
    //1. MemberRepository 인터페이스 의존
    private final MemberRepository memberRepository;

    //2. throws SQLException 생략
    @Transactional
    public void accountTransfer(String fromId, String toId, int money) {
        bizLogic(fromId, toId, money);
    }

    private void bizLogic(String fromId, String toId, int money) {
        //...
    }
}
```

## 4. 예외 복구
상황: DB에 hello라는 중복된 아이디가 있는 경우 hello1111와 같이 임의의 숫자를 붙여서 가입하는 경우

<img src="https://user-images.githubusercontent.com/108064146/219694060-c577ee64-08f3-440a-ab7f-a4de2cfca7c3.png">

### 4.1 예외 복구 과정
1. 데이터를 DB에 저장할 때 같은 ID가 이미 DB에 있다면, DB는 오류 코드를 반환
2. 오류 코드를 받은 JDBC드라이버는 SQLException에 오류 코드(errorCode)를 포함하여 던진다.
3. 리포지토리에서 예외 코드로 어떤 예외인지 구분하여 새로운 런타임 예외를 만들어 서비스 계층에 던진다.

- `MyDbException`런타임 예외를 상속받아 `MyDuplicateKeyException`런타임 예외를 구현함으로써 DB에 관련된 예외라는 의미있는 계층을 형성한다.

### 4.2 예외 복구 구현

```java
@Slf4j
public class ExTranslatorV1Test {

    Repository repository;
    Service service;

    @BeforeEach
    void init() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);
        repository = new Repository(dataSource);
        service = new Service(repository);
    }

    @Test
    void duplicateKeySave() {
        service.create("myId");
        service.create("myId");
    }

    @RequiredArgsConstructor
    static class Service {

        private final Repository repository;

        public void create(String memberId) {
            try {
                repository.save(new Member(memberId, 10000));
                log.info("saveId={}", memberId);
            } catch (MyDuplicateKeyException e) { // 4. 중복으로 인해 발생한 예외의 경우 
                log.info("키 중복, 복구 시도");
                String retryId = generateNewId(memberId); // 5. 새로운 아이디를 생성
                log.info("retryId={}", retryId);
                repository.save(new Member(retryId, 10000)); // 6. 해당 새로운 아이디를 DB에 저장
            }
        }

        private String generateNewId(String memberId) {
            return memberId + new Random().nextInt(10000);
        }
    }

    @RequiredArgsConstructor
    static class Repository {

        private final DataSource dataSource;

        public Member save(Member member) {
            String sql = "insert into member(member_id, money) values(?, ?)";

            Connection con = null;
            PreparedStatement pstmt = null;

            try {
                con = dataSource.getConnection();
                pstmt = con.prepareStatement(sql);
                pstmt.setString(1, member.getMemberId());
                pstmt.setInt(2, member.getMoney());
                pstmt.executeUpdate();
                return member;
            } catch (SQLException e) {
                if (e.getErrorCode() == 23505) { // 1. 예외 코드가 중복을 나타나는 23505인 경우 
                    throw new MyDuplicateKeyException(e); // 2. 새로운 런타임 예외를 생성해서 던진다.
                }
                throw new MyDbException(e); // 3. 그 밖의 경우 MyDbException예외를 던진다.
            } finally {
                JdbcUtils.closeStatement(pstmt);
                JdbcUtils.closeConnection(con);
            }
        }
    }
}
```
- 복구할 수 없는 예외(MyDbException)는 공통으로 예외를 처리하는 곳에서 예외 로그를 남기는 것이 좋다.
- 항상 예외에서 예외 코드를 확인해서 런타임 예외를 만들어내는 것은 현실성이 없다. (`예외 변환기` 사용)
## 5. 스프링 데이터 접근 예외
스프링은 데이터 접근과 관련된 예외를 추상화하여 제공한다.
<img src="https://user-images.githubusercontent.com/108064146/219708945-7e31bc81-d46a-40ad-a8d2-6ef898d16b78.png">

- 최고 상위 예외는 `DataAccessException`이며 런타임 예외를 구현한다. 크게 `NonTransient`예외와 `Transient`예외로 구분한다.
- `Transient`(일시적 O): `Transient`하위 예외는 동일한 SQL을 다시 시도했을 때 성공할 가능성이 있는 경우의 예외이다.
    + ex. 쿼리 타임아웃, 락 오류
- `NonTransient`(일시적 X): `NonTransient`하위 예외는 같은 SQL을 다시 시도했을 때 성공할 가능성이 없는 경우의 예외이다. 
    + ex. SQL 문법 오류, DB 제약조건 위배

## 6. 스프링이 제공하는 예외 변환기
스프링은 DB에서 발생하는 오류 코드를 확인하여 스프링이 정의한 예외로 자동으로 변환해주는 변환기를 제공한다.
- `exTranslator.translate(읽을 수 있는 설명, 실행한 sql, 발생된 예외)`
- `org.springframework.jdbc.support.sql-error-codes.xml`에서 예외 코드에 맞는 에러를 찾아 반환한다.
- 서비스 계층에서 예외를 잡아서 복구해야 하는 경우, 예외가 스프링이 제공하는 데이터 접근 예외로 변경되어서 서비스 계층에 넘어오기 때문에 필요한 경우 예외를 잡아서 복구하면된다.
```java
/**
 * SQLExceptionTranslator 추가
 */
@Slf4j
public class MemberRepositoryV4_2 implements MemberRepository {

    private final DataSource dataSource;
    private final SQLExceptionTranslator exTranslator;

    public MemberRepositoryV4_2(DataSource dataSource) {
        this.dataSource = dataSource;
        // error코드를 기반으로한 예외 변한기를 생성
        this.exTranslator = new SQLErrorCodeSQLExceptionTranslator(dataSource);
    }

    //...

    @Override
    public Member findById(String memberId) {
        String sql = "select * from Member where member_id = ?";

        Connection con = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;

        try {
            con = getConnection();
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
            throw new MyDbException(e);
        } finally {
            close(con, pstmt, rs);
        }
    }

    //...
}
```

## 7. JDBC 반복 문제 해결

### 7.1 중복되는 반복
- 커넥션 조회, 커넥션 동기화
- PreparedStatement 생성 및 파리미터 바인딩
- 쿼리 실행
- 결과 바인딩
- 예외 발생시 스프링 예외 변환기 실행
- 리소스 종료

### 7.2 템플릿 콜백 패턴을 이용 - JdbcTemplate
스프링은 템플릿 콜백 패턴을 이용한 JdbcTemplate을 제공한다.
- 트랜잭션을 위한 커넥션 동기화, 스프링 예외 변환기도 자동으로 실행해준다.
```java
@Slf4j
public class MemberRepositoryV5 implements MemberRepository {

    private final JdbcTemplate template;

    public MemberRepositoryV5(DataSource dataSource) {
        template = new JdbcTemplate(dataSource); // JdbcTemplate을 생성
    }

    @Override
    public Member save(Member member) {
        String sql = "insert into member(member_id, money) values (?, ?)";
        template.update(sql, member.getMemberId(), member.getMoney()); // sql, ?, ?
        return member;
    }

    @Override
    public Member findById(String memberId) {
        String sql = "select * from Member where member_id = ?";
        // 1. 커넥션 실행 and 파라미터 바인딩 and 쿼리 실행
        // 2. 검색 결과를 ResultSet에 저장
        // 3. ResultSet에 데이터가 있으면 객체로 생성해서 반환
        return template.queryForObject(sql, memberRowMapper(), memberId);
    }

    @Override
    public void update(String memberId, int money) {
        String sql = "update member set money = ? where member_id = ?";
        template.update(sql, money, memberId);
    }

    @Override
    public void delete(String memberId) {
        String sql = "delete from member where member_id = ?";
        template.update(sql, memberId);
    }

    
    private RowMapper<Member> memberRowMapper() {
        return (rs, rowNum) -> {
            Member member = new Member();
            member.setMemberId(rs.getString("member_id"));
            member.setMoney(rs.getInt("money"));
            return member;
        };
    }
}
```

## Reference
> 스프링 DB 1편 - 데이터 접근 핵심 원리 [김영한]

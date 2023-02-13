# JDBC 이해와 간단한 CRUD 구현

## 1. 서버와 DB 연결
1. `커넥션 연결`: 주로 TCP/IP를 사용해서 커넥션을 연결
2. `SQL 전달`: 애플리케이션 서버는 DB가 이해할 수 있는 SQL을 연결된 커넥션을 통해 DB에 전달
3. `결과 응답`: DB는 전달된 SQL을 수행하고 그 결과를 응답

## 2. JDBC
각각의 DB마다 접근 방법이 모두 달랐지만 JDBC 인터페이스의 등장으로 JDBC 인터페이스 1개로만 접근하면 된다.
- `JDBC 드라이버`: DB회사에서 자신의 DB에 맞도록 구현해서 라이브러리로 제공하는 구현체 (ex. MySQL JDBC 드라이버)
    + Connection, Statement, ResultSet 모두 구현되어 있다.

<img src = "https://user-images.githubusercontent.com/108064146/218424282-4a22c535-5668-4cd7-8f5b-777105070857.png">

### 2.1 제공하는 기능
- `java.sql.Connection` - 연결
- `java.sql.Statement` - SQL을 담은 내용
- `java.sql.ResultSet` - SQL 요청 응답

## 3. SQL Mapper, JPA
최근에는 JDBC를 직접 사용하기 보다는 SQL Mapper와 JPA 등을 이용해 JDBC를 사용한다.

### 3.1 SQL Mapper
- JDBC 편리하게 사용
- SQL 응답 결과를 객체로 편리하게 변환
- JDBC의 반복 코드를 제거
- 개발자가 SQL을 직접 작성
- ex. JdbcTemplate, MyBatis

### 3.2 ORM
객체를 관계형 DB 테이블과 매핑해주는 기술
- SQL을 직접 작성하지 않고 ORM 기술이 동적으로 SQL을 만들어 실행해준다.
- 각각의 DB 마다 다른 SQL을 사용해야하는 문제도 해결해준다.
- ex. JPA, Hibernate, EclipseLink

### 3.3 SQL Mapper vs ORM
- `SQL Mapper`: SQL만 직접 작성하면 나머지 번거로운 부분은 SQL Mapper가 대신 해결해주며 배우기 쉽다.
- `ORM`: SQL 자체를 작성하지않아 개발 생산성이 매우 높아지지만 쉬운 기술이 아니라 깊이 있는 학습이 필요하다.

## 4. DB에 커넥션 요청
- `DriverManager.getConnection(URL, USERNAME, PASSWORD)`: 해당 URL, USERNAME, PASSWORD에 맞는 DB 드라이버의 connection 구현체를 반환 (checked 예외를 발생시킬 수 있기 때문에 예외 처리 필수)
- 
```java
// URL: "jdbc:h2:tcp://localhost/~/test"
// USERNAME: "sa"
// PASSWORD: ""
@Slf4j
public class DBConnectionUtil {

    public static Connection getConnection() {
        try {
            Connection connection = DriverManager.getConnection(URL, USERNAME, PASSWORD);
            return connection;
        } catch (SQLException e) {
            throw new IllegalStateException();
        }
    }
}
```

## 5. 등록 with JDBC
```java
@Slf4j
public class MemberRepositoryV0 {

    public Member save(Member member) throws SQLException{
        String sql = "insert into member(member_id, money) values (?, ?)"; // 1. sql 정의

        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            con = getConnection();  // 2. 커넥션 가져오기
            pstmt = con.prepareStatement(sql); // 3. ?를 통한 파리미터 바인딩을 가능하게 해준다. (SQL Injection 공격 예방)
            pstmt.setString(1, member.getMemberId()); // 4. 해당 sql의 ?에 setXXX타입의 값을 매핑 (?위치, 값)
            pstmt.setInt(2, member.getMoney());
            pstmt.executeUpdate();  // 5. DB에 sql 실행 (반환 값으로 영항받은 열의 갯수 반환)
            return member;
        } catch(SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            close(con, pstmt, null); // 6. 사용한 리소스 정리. 즉, 연결되어 있는 커넥션을 끊기. (항상 실행한 역순으로 닫아준다.)
                                     // 커넥션이 계속 유지되면 리소스 누수가 발생하여 커녁션 부족으로 장애가 발생할 수 있다.
        }

    }

    private void close(Connection con, Statement stmt, ResultSet rs) {

        if (rs != null) {
            try {
                rs.close();
            } catch (SQLException e) {
                log.info("error", e);
            }

        }
        if (stmt != null) {
            try {
                stmt.close();
            } catch (SQLException e) {
                log.info("error", e);
            }
        }

        if (con != null) {
            try {
                con.close();
            } catch (SQLException e) {
                log.info("error", e);
            }
        }
    }

    private static Connection getConnection() {
        return DBConnectionUtil.getConnection();
    }
}
```

## 6. 조회 with JDBC

```java
public Member findById(String memberId) throws SQLException {
        String sql = "select * from Member where member_id = ?";

        Connection con = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null; // 쿼리의 결과물을 담는 객체

        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, memberId);
            rs = pstmt.executeQuery(); // 쿼리 실행하고 결과물 반환

            if (rs.next()) { // 내부의 커서(cursor)를 이동해서 다음 데이터 조회. 첫 커서는 값이 아닌 속성을 가르킨다. (값이 있으면: true, 없으면: false)
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
            close(con, pstmt, rs);
        }
    }

```

## 7. 수정 with JDBC

```java
public void update(String memberId, int money) throws SQLException {
        String sql = "update member set money = ? where member_id = ?";
        Connection con = null;
        PreparedStatement pstmt = null;
        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setInt(1, money);
            pstmt.setString(2, memberId);
            pstmt.executeUpdate();
        } catch (SQLException e) {
            log.info("db error", e);
            throw e;
        } finally {
            close(con, pstmt, null);
        }
    }
```

## 8. 삭제 with JDBC

```java
public void delete(String memberId) throws SQLException {
        String sql = "delete from member where member_id = ?";
        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, memberId);
            pstmt.executeUpdate();
        } catch (SQLException e) {
            log.info("db error", e);
            throw e;
        } finally {
            close(con, pstmt, null);
        }
    }
```

## 9. CRUD Test
- 같은 테스트를 반복할 수 있는 Test가 좋은 코드이지만 해당 Test의 경우 중간에 오류가 발생하면 delete를 수행할 수 없어 반복할 수 없으므로 좋은 코드는 아니다. (트랜잭션을 이용해서 해결 가능)
```java
@Slf4j
class MemberRepositoryV0Test {

    MemberRepositoryV0 repository = new MemberRepositoryV0();

    @Test
    void crud() throws SQLException {
        //save
        Member member = new Member("member5", 10000);
        repository.save(member);

        //findById
        Member findMember = repository.findById(member.getMemberId());
        log.info("findMember = {}", findMember);
        log.info("findMember.equals(member) = {}", findMember.equals(member));
        assertThat(findMember).isEqualTo(member);

        //update
        repository.update(member.getMemberId(), 20000);
        Member updatedMember = repository.findById(member.getMemberId());
        assertThat(updatedMember.getMoney()).isEqualTo(20000);

        //delete
        repository.delete(member.getMemberId());
        assertThatThrownBy(() -> repository.findById(member.getMemberId()))
                .isInstanceOf(NoSuchElementException.class);
    }
}
```

## Reference
> 스프링 DB 1편 - 데이터 접근 핵심 원리 [김영한]

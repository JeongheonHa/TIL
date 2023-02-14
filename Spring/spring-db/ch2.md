# Connection Pool

## 1. 커넥션 요청 과정
<img src = "https://user-images.githubusercontent.com/108064146/218515553-e0feb5ea-7eb4-4019-bc07-22fcc16febcb.png">

- 이렇게, DB에서 복잡한 과정이 발생하기 때문에 시간이 많이 걸린다.
- 애플리케이션 서버에서도 `TCP/IP`커넥션을 새로 생성하기 위한 리소스를 매번 사용해야하기 때문에 고객이 애플리케이션을 사용할 때 SQL을 실행하는 시간 뿐만 아니라 커넥션을 새로 만드는 시간이 추가되기 때문에 응답 속도가 느려진다.

## 2. Connection Pool
커넥션 풀에 DB와 `TCP/IP`로 연결되어있는 커넥션들을 보관해둠으로써 시간을 많이 잡아먹는 커넥션 요청 과정을 하지 않는다.

### 2.1 커넥션 풀 초기화
<img src = "https://user-images.githubusercontent.com/108064146/218515013-38fbe7b8-2519-403f-83b4-eec1193aac5f.png">

### 2.2 커넥션 풀 사용
<img src = "https://user-images.githubusercontent.com/108064146/218515870-c06fc5c0-d7e2-40a4-94c8-1ce5788e8373.png">

- 기본 값으로 보통 `10`개가 보관된다.
- TCP/IP로 DB와 커넥션이 연결되어 있는 상태이기 때문에 언제든지 SQL을 DB에 전달 가능
- 커넥션을 모두 사용하고 나면 커넥션을 종료하는 것이 아닌, 연결된 상태의 커넥션을 그대로 커넥션 풀에 반환
- 커넥션을 요청할 때는 `proxyConnection`객체를 생성해서 커넥션을 담아 사용하고 `close`가 실행되면 proxyConnection은 버려지고 안에 있는 커넥션은 커넥션 풀로 반환
- 커넥션 풀의 최대 커넥션 갯수는 서비스의 특징, 서버 스펙, DB 서버 스펙에 따라 다르기 때문에 성능 테스트를 통해 정한다.
- 서버당 최대 커넥션 수를 제한하여 DB에 무한정 연결이 생성되는 것을 막기 때문에 DB를 보호하는 효과가 있다.
- 보통 커넥션 풀로 오픈 소스인 `hikariCP`를 사용한다. (스프링 부트가 제공)

## 3. DataSource
커넥션을 얻는 방법을 추상화 해주는 자바 표준 인터페이스

<img scr = "https://user-images.githubusercontent.com/108064146/218517409-80272d8b-5ed5-47dc-831a-46d283519a16.png">

```java
public interface DataSource {
    Connection getConnection() throws SQLException;
}
```
- 커넥션을 얻기 위해 DataSource만 의존하면 된다.
- DriverManager는 DataSource를 사용하지 않기 때문에 DriverManager를 직접 사용해야한다. 따라서 스프링은 DataSource를 구현한 `DriverManagerDataSource`라는 커넥션을 생성해서 반환해주는 구현체를 제공한다.


## 4. DriverManager vs DriverManagerDataSource
- 설정과 사용의 분리: 커넥션 생성에 필요한 설정 정보를 DataSource가 생성되는 시점에 미리 넣어두어 DataSource를 사용하는 쪽(리포지토리)에서는 `dataSource.getConnection()`만 호출하면 된다.
```java
@Slf4j
public class ConnectionTest {

    @Test
    void driverManager() throws SQLException {
        Connection con1 = DriverManager.getConnection(URL, USERNAME, PASSWORD); // 커넥션 설정과 사용이 동시에 이루어진다.
        Connection con2 = DriverManager.getConnection(URL, USERNAME, PASSWORD);
        log.info("connection = {}, class = {}", con1, con1.getClass());
        log.info("connection = {}, class = {}", con2, con2.getClass());
    }

    @Test
    void driverManagerDataSource() throws SQLException {
        DataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD); // 커넥션 설정과 사용이 분리되어 있다.
        useDataSource(dataSource);  // 사용
    }

    private static void useDataSource(DataSource dataSource) throws SQLException {
        Connection con1 = dataSource.getConnection();
        Connection con2 = dataSource.getConnection();
        log.info("connection = {}, class = {}", con1, con1.getClass());
        log.info("connection = {}, class = {}", con2, con2.getClass());
    }
}
```

## 5. 데이터 소스 커넥션 풀 추가
- 커넥션 풀에서 커넥션 생성은 애플리케이션 실행 속도에 영향을 주지 않기 위해 별도의 쓰레드를 사용하기 때문에 `Thread.sleep`을 통해 test 쓰레드를 대기시켜야 쓰레드 풀에 커넥션이 생성되는 로그를 확인할 수 있다.
```java
@Test
void dataSourceConnectionPool() throws SQLException, InterruptedException {
    //커넥션 풀링: HikariProxyConnection(Proxy) -> JdbcConnection(Target) 
    HikariDataSource dataSource = new HikariDataSource(); 
    dataSource.setJdbcUrl(URL);
    dataSource.setUsername(USERNAME);
    dataSource.setPassword(PASSWORD);
    dataSource.setMaximumPoolSize(10); // 커넥션 풀 최대 사이즈 지정
    dataSource.setPoolName("MyPool");  // 커넥션 풀 이름 지정
    useDataSource(dataSource);
    Thread.sleep(1000); //커넥션 풀에서 커넥션 생성 시간 대기
}
```

## 6. 데이터 소스 사용


### 6.1 의존관계 주입
DataSource는 의존관계 주입을 통해 받아온다.
```java
@Slf4j
public class MemberRepositoryV1 {

    private final DataSource dataSource;

    public MemberRepositoryV1(DataSource dataSource) {
        this.dataSource = dataSource;
    }
}
```

### 6.2 close
JdbcUtils를 이용해서 close를 한다.
```java
private void close(Connection con, Statement stmt, ResultSet rs) {
    JdbcUtils.closeConnection(con);
    JdbcUtils.closeStatement(stmt);
    JdbcUtils.closeResultSet(rs);
}
```

### 6.3 test

```java
@Slf4j
class MemberRepositoryV1Test {

    MemberRepositoryV1 repository;

    @BeforeEach
    void beforeEach() throws Exception {
        //항상 새로운 커넥션 획득
        //DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);

        //커넥션 풀링
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl(URL);
        dataSource.setUsername(USERNAME);
        dataSource.setPassword(PASSWORD);

        repository = new MemberRepositoryV1(dataSource);
    }

    @Test
    void crud() throws SQLException {

    }
}
```

## Reference
> 스프링 DB 1편 - 데이터 접근 핵심 원리 [김영한]

# Transaction problems

## 1. 트랜잭션 문제
트랜잭션을 적용하면 생기는 문제들

### 1.1 JDBC 구현 기술이 서비스 계층에 누수되는 문제
Service 계층은 순수 비즈니스 로직만 존재해야 유지 보수가 좋다. 하지만, 트랜잭션을 적용하기 위해서는 JDBC 구현 기술을 Service 계층에서 의존해야한다.

### 1.2 트랜잭션 동기화 문제
같은 트랜잭션을 유지하기 위해 커넥션을 파라미터로 넘기면 같은 기능을 트랜잭션 용 기능과 트랜잭션을 유지하지 않아도 되는 기능으로 분리해야한다. (불필요한 반복, 코드 길어짐)

### 1.3 트랜잭션 적용 반복 문제
트랜잭션 적용 코드를 보면 반복이 많다. (try - catch - finally)

## 2. 예외 누수 문제
데이터 접근 계층의 JDBC 구현 기술 `예외`가 서비스 계층으로 전파된다. (ex. SQLException)
- SQLExcetion은 JDBC 기술이기 때문에 다른 데이터 접근 기술로 바꿀 경우 해당 기술의 예외로 서비스 계층도 수정해야한다.

## 3. JDBC 반복 문제
순수 JDBC의 기술을 이용해서 데이터를 접근하다 보니 try, catch, finally, PrepareStatement 등 반복되는 코드가 많다.

## 4. 트랜잭션 문제 해결

### 4.1 JDBC 구현 기술이 서비스 계층에 누수되는 문제 해결 - 트랜잭션 추상화
서비스 계층에서 트랜잭션을 시작하기 위해 의존했던 JDBC 기술 대신 트랜잭션 기능들을 추상화 시킴으로써 데이터 접근 기술에 대한 의존 관계를 제거한다.

#### 스프링의 트랜잭션 추상화
사용하는 데이터 접근 기술의 트랜잭션 기능을 제공하는 구현체(트랜잭션 매니저)를 주입하면 된다.
- `JdbcTransactionManager`: DataSourceTransactionManager를 상속받아 기능을 좀 더 확장한 구현체 (기능의 차이는 크지 않다.)

<img src="https://user-images.githubusercontent.com/108064146/219557252-7dd70c03-bc91-4532-9c60-f1e4e2791d20.png">

#### PlatformTransactionManager Interface
- `getTransaction()`: 트랜잭션 시작 (이미 진행 중인 트랜잭션이 있는 경우 해당 트랜잭션에 참여 가능)
```java
package org.springframework.transaction;

public interface PlatformTransactionManager extends TransactionManager {

    TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException;
    void commit(TransactionStatus status) throws TransactionException;
    void rollback(TransactionStatus status) throws TransactionException;
}
```

### 4.2 트랜잭션 동기화 문제 해결 - 트랜잭션 동기화 매니저
커넥션이 필요하면 트랜잭션 동기화 매니저를 통해 커넥션을 얻는다.
- 쓰레드 로컬을 사용해 커넥션을 동기화하기 때문에 멀티 쓰레드 환경에서 커넥션을 안전하게 보관할 수 있다.

#### 트랜잭션 시작과 로직 수행 과정

<img src="https://user-images.githubusercontent.com/108064146/219566789-331ccaf9-76ee-456b-b787-9a064d3113ab.JPG">

1. 서비스 계층에서 트랜잭션 매니저에게 트랜잭션을 시작하라고 요청
2. 트랜잭션 매니저는 트랜잭션을 시작하기위해 주입받은 데이터 소스를 사용해 커넥션을 생성
3. 커넥션을 수동 커밋 모드로 변경해서 트랜잭션을 시작
4. 커넥션을 트랜잭션 동기화 매니저에 보관
5. 트랜잭션 동기화 매니저는 쓰레드 로컬에 커넥션을 보관
6. 서비스 계층에서 리포지토리의 메서드를 호출
7. 리포지토리는 데이터에 접근하기 위해 트랜잭션 동기화 매니저에서 커넥션을 꺼내서 사용한다. (커넥션 유지)
8. 획득한 커넥션을 사용해 SQL을 DB에 전달

#### 트랜잭션 종료 과정
<img src="https://user-images.githubusercontent.com/108064146/219571284-6a1edb84-ccf4-4ade-abc9-ea498c28dc7b.png">

9. 비즈니스 로직이 끝나고 서비스 계층에서 `commit` 또는 `rollback`으로 트랜잭션 매니저에게 트랜잭션을 종료해달라고 요청
10. 트랜잭션을 종료하기 위해 트랜잭션 동기화 매니저에서 커넥션을 가져온다.
11. 가져온 커넥션은 통해 `commit` 또는 `rollback`
12. 전체 리소스 정리 (자동 커밋 모드로 변경, `con.close()`로 커넥션 종료 or 커넥션 풀에 반납)

> 쓰레드 로컬 사용시 각각의 쓰레드마다 별도의 저장소가 부여되어 해당 쓰레드만이 해당 데이터에 접근할 수 있다.

### 4.3 트랜잭션 매니저 적용

#### 트랜잭션 동기화 매니저에서 커넥션 가져오기
- `DataSourceUtils.getConnection()`: 트랜잭션 동기화 매니저가 관리하는 커넥션이 있으면 해당 커넥션을 반환한다. (없으면 새로운 커넥션 생성해서 반환)
- `DataSourceUtils.releaseConnection()`: 트랜잭션 동기화 매니저가 관리하는 커넥션이면 커넥션을 닫지 않고 트랜잭션이 종료될 때까지 유지한다. (아니면 커넥션을 닫는다.)

```java
@Slf4j
public class MemberRepositoryV3 {

    //...

    private void close(Connection con, Statement stmt, ResultSet rs) {
        DataSourceUtils.releaseConnection(con, dataSource);
        JdbcUtils.closeStatement(stmt);
        JdbcUtils.closeResultSet(rs);
    }

    private Connection getConnection() throws SQLException {
        Connection con = DataSourceUtils.getConnection(dataSource);
        log.info("get connection={}, class={}", con, con.getClass());
        return con;
    }
}
```

#### 트랜잭션 매니저를 이용해 트랜잭션 시작
- `transactionManager.getTransaction(new DefaultTransactionAttribute())`: 트랜잭션을 시작하고 현재 트랜잭션의 상태 정보를 반환
```java
@Slf4j
@RequiredArgsConstructor
public class MemberServiceV3_1 {

    private final PlatformTransactionManager transactionManager;
    private final MemberRepositoryV3 memberRepository;

    public void accountTransfer(String fromId, String toId, int money) throws SQLException {
        //트랜잭션 시작
        TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionAttribute());

        try {
            bizLogic(fromId, toId, money);
            transactionManager.commit(status);
        } catch (Exception e) {
            transactionManager.rollback(status);
            throw new IllegalStateException(e);
        }
    }

    //...
}
```

#### test

```java
class MemberServiceV3_1Test {

    private MemberRepositoryV3 memberRepository;
    private MemberServiceV3_1 memberService;

    //...

    @BeforeEach
    void before() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager(dataSource);
        memberRepository = new MemberRepositoryV3(dataSource);
        memberService = new MemberServiceV3_1(transactionManager, memberRepository);
    }

    //...
}
```

### 4.3 트랜잭션 적용 반복 문제 해결 - 트랜잭션 템플릿
`템플릿 콜백 패턴`을 활용하여 반복 문제를 해결
- 스프링이 제공하는 TransactionTemplate라는 템플릿 클래스를 사용

#### TransactionTemplate 인터페이스
```java
public class TransactionTemplate {
    private PlatformTransactionManager transactionManager;
    public <T> T execute(TransactionCallback<T> action){..} //응답 값이 있을 때
    void executeWithoutResult(Consumer<TransactionStatus> action){..} // 응답 값이 없을 때
}
```

#### 트랜잭션 템플릿 적용
템플릿을 사용함으로써 트랜잭션을 시작하고 종료하는 코드를 템플릿이 대신 수행해준다.
```java
@Slf4j
public class MemberServiceV3_2 {

    private final TransactionTemplate txTemplate;
    private final MemberRepositoryV3 memberRepository;

    public MemberServiceV3_2(PlatformTransactionManager transactionManager, MemberRepositoryV3 memberRepository) {
        this.txTemplate = new TransactionTemplate(transactionManager); // 템플릿에 트랜잭션 매니저를 주입받는다.
        this.memberRepository = memberRepository;
    }

    public void accountTransfer(String fromId, String toId, int money) throws SQLException {
        txTemplate.executeWithoutResult((status) -> { // 템플릿을 이용해서 로직 수행
            try {
                bizLogic(fromId, toId, money);
            } catch (SQLException e) {
                throw new RuntimeException(e);
            }
        });
    }

    //...
}
```

### 4.4 남아있는 문제점
Service 계층에는 핵심 비즈니스 로직만이 존재해야하지만 여전히 트랜잭션과 관련된 코드(txTemplate)가 남아있다.
- 트랜잭션 AOP로 해결

### 4.5 트랜잭션 AOP
스프링 AOP를 통해 프록시를 도입하여 해결한다.
- 트랜잭션 프록시가 트랜잭션 처리 로직을 모두 가져가 트랜잭션을 시작한 후 실제 서비스를 대신 호출한다.
- 스프링은 트랜잭션 AOP를 처리하기 위한 모든 기능을 제공하며 스프링 부트를 사용하면 트랜잭션 AOP를 처리하기 위해 필요한 스프링 빈들도 자동으로 등록해준다.
- 개발자는 트랜잭션 처리가 필요한 곳에 `@Transactional` 애노테이션만 붙여주면 스프링의 트랜잭션 AOP가 해당 애노테이션을 인식하여 트랜잭션 프록시를 적용해준다.

<img src="https://user-images.githubusercontent.com/108064146/219576603-19ca8914-9513-4a1b-8782-25b99ceff019.png">

#### 트랜잭션 프록시 코드 예시

```java
public class TransactionProxy {

    private MemberService target;

    public void logic() { 
        //트랜잭션 시작
        TransactionStatus status = transactionManager.getTransaction(..);
        try {
        //실제 대상 호출 
        target.logic();
        transactionManager.commit(status); //성공시 커밋 
        } catch (Exception e) {
            transactionManager.rollback(status); //실패시 롤백
            throw new IllegalStateException(e);
        }
    } 
}
```

#### 서비스 계층 코드

```java
public class Service {
    public void logic() {
        //트랜잭션 관련 코드 제거, 순수 비즈니스 로직만 남음
        bizLogic(fromId, toId, money);
    }
}
```

#### 트랜잭션 AOP 적용
- `@Transactional`는 메서드에 붙여도 되고, 클래스에 붙여도 된다. 클래스에 붙일 경우 public 메서드가 AOP 적용 대상이 된다.

```java
@RequiredArgsConstructor
public class MemberServiceV3_3 {

    private final MemberRepositoryV3 memberRepository;

    @Transactional
    public void accountTransfer(String fromId, String toId, int money) throws SQLException {
        bizLogic(fromId, toId, money);
    }
}
```

#### 트랜잭션 AOP test
- 스프링 부트는 `DataSource`를 스프링 빈에 자동으로 등록한다. (빈 이름: dataSource)
- `application.properties`에 있는 속성을 사용해서 DataSource를 생성한다.
- spring.datasource.url 속성이 없으면 내장 DB(메모리 DB)를 생성하려고 시도한다.
- 스프링 부트는 적절한 `트랜잭션 매니저`를 자동으로 스프링 빈에 등록한다. (빈 이름: transactionManager)
- DataSource: HikariDataSource
- 트랜잭션 매니저: 현재 등록된 라이브러리를 보고 등록
```html
//application.properties
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.username=sa
spring.datasource.password=
```
```java
@Slf4j
@SpringBootTest // 스프링 AOP를 적용하기위해 스프링을 띄운다.
class MemberServiceV3_3Test {

    @Autowired
    private MemberRepositoryV3 memberRepository;
    @Autowired
    private MemberServiceV3_3 memberService;

    @TestConfiguration // 테스트 내에서 내부 설정 클래스를 만들어 사용할 때 사용 (스프링 빈으로 등록)
    static class TestConfig {
        
        private final DataSource dataSource;

        public TestConfig(DataSource dataSource) {
            this.dataSource = dataSource;
        }

        @Bean
        MemberRepositoryV3 memberRepositoryV3() {
            return new MemberRepositoryV3(dataSource());
        }

        @Bean
        MemberServiceV3_3 memberServiceV3_3() {
            return new MemberServiceV3_3(memberRepositoryV3());
        }
    }

    @Test
    @DisplayName("Proxy 객체 확인")
    void AopCheck() {
        log.info("memberService class={}", memberService.getClass());
        log.info("memberRepository class={}", memberRepository.getClass());
        assertThat(AopUtils.isAopProxy(memberService)).isTrue();
        assertThat(AopUtils.isAopProxy(memberRepository)).isFalse();
    }

    //...
}
```

## 5. 전체 흐름

<img src="https://user-images.githubusercontent.com/108064146/219612951-c3a198e5-4013-4ced-82e9-ab2d13b8de5b.png">

## Reference
> 스프링 DB 1편 - 데이터 접근 핵심 원리 [김영한]

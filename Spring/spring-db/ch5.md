# Exception

## 1. 기본 예외

### 1.1 예외의 기본 규칙 2가지
1. 예외는 잡아서 처리하거나 던져야 한다.
2. 예외를 잡거나 던질 때 지정한 예외 뿐만 아니라 그 예외의 자식들도 함께 처리된다.

### 1.2 예외를 계속 던질 경우
- `자바 main()쓰레드의 경우`: 예외 로그를 출력하면서 시스템이 종료된다.
- `웹 애플리케이션의 경우`: WAS가 해당 예외를 받아서 처리, 주로 오류페이지를 보여준다.

### 1.3 예외 팁
- catch를 예외를 잡을 때는 구체적으로 지정해주는 것이 좋다.
- throws로 예외를 던질 때는 구체적으로 지정해주는 것이 좋다.

## 2. 체크 예외

### 2.1 체크 예외의 기본 원칙
1. 기본적으로 런타임 예외를 사용하자
2. 체크 예외는 비즈니스 로직상 의도적으로 던지는 예외에만 사용하자
    + 그렇지 않더라도 체크 예외를 런타임 예외로 만들어 Document에 잘 정의해도 된다.

### 2.2 체크 예외의 문제점
- Repository: DB에 접근해서 데이터를 저장하고 관리한다. (SQLException 체크 예외 발생)
- NetworkClient: 외부 네트워크에 접속해서 어떤 기능을 처리하는 객체이다. (ConnectException 체크 예외 발생)

<img src="https://user-images.githubusercontent.com/108064146/219634173-0eb5dcdf-9086-41d6-8dbf-a9b443aa944b.png">

#### 복구 불가능한 예외
- ConnectException처럼 연결이 실패하거나, SQLException처럼 데이터베이스에서 발생하는 문제처럼 심각한 문제들은 대부분 애플리케이션 로직에서 처리할 방법이 없다. (서비스, 컨트롤러에서는 해결할 수 없다.)
- 오류 로그를 남기고 개발자가 해당 오류를 빠르게 인지하는 것이 필요하다.
- 웹 애플리케이션이라면 서블릿의 오류 페이지나, 또는 스프링 MVC가 제공하는 `ControllerAdvice`에서 이런 예외를 공통으로 처리한다.
- API라면 HTTP 상태코드 500을 사용해서 응답을 내려준다.

#### 의존 관계에 대한 문제
- 체크 예외이기 때문에 컨트롤러나 서비스 입장에서는 본인이 처리할 수 없어도 throws를 통해 던지는 예외를 선언해야하므로 SQLException이라는 JDBC 기술에 의존하게 된다.
- 따라서 데이터 접근 기술을 바꾼다면 해당 예외를 던지는 모든 부분을 수정해야한다.

### 2.3 throws Exception를 선언하는 경우
최상위 타입의 예외를 던진다면 다른 체크 예외를 체크할 수 있는 기능이 무효화 되고, 중요한 체크 예외를 다 놓치게 된다.

## 3. 런타임 예외
- SQLException 체크 예외 -> RuntimeSQLException 런타임 예외
- ConnectException 체크 예외 -> RuntimeConnectException 런타임 예외

<img src="https://user-images.githubusercontent.com/108064146/219641344-90013deb-deb3-4b1f-a111-3f9ef20f689a.png">

### 3.1 체크 예외의 문제점 해결
- 복구 불가능한 예외: 런타임 예외를 사용하면 웹 애플리케이션에서 해당 예외를 신경 쓰지않고 한번에 공통으로 처리하면 된다.
- 의존 관계에 대한 문제: 런타임 예외는 throws를 생략할 수 있어 의존하지 않아도 된다.
- 데이터 접근 기술이 바뀌어도 예외를 공통으로 처리하는 한 곳만 수정하면 된다.

### 3.2 런타임 예외 문서화
- 런타임 예외는 예외를 인지할 수 있도록 `문서화` 하거나 `throws 런타임예외`를 남겨주는 것이 좋다.

```java
// 예시
/**
 * Make an instance managed and persistent.
 * @param entity  entity instance
 * @throws EntityExistsException if the entity already exists.
 * @throws IllegalArgumentException if the instance is not an
 *         entity
 * @throws TransactionRequiredException if there is no transaction when
 *         invoked on a container-managed entity manager of that is of type
 *         <code>PersistenceContextType.TRANSACTION</code>
 */
public void persist(Object entity);
```

## 4. 예외 포함과 스택 트레이스

### 4.1 예외 포함
예외를 전환할 때는 무조건 기존 예외를 원인 예외로 포함해야한다.
- 원인 예외로 포함시켜야만 stacktrace로 확인할 수 있다.
```java
public void call() {
    try {
        runSQL();
    } catch (SQLException e) {
        throw new RuntimeSQLException(e); //기존 예외(e) 포함 
    }
}
```

### 4.2 스택 트레이스
- `log.info("예외 처리, message={}", e.getMessage(), e);`: `{}`는 예외 메세지를 출력하고, stacktrace는 예외만 파라미터로 넣어주면 된다.

## Reference
> 스프링 DB 1편 - 데이터 접근 핵심 원리 [김영한]

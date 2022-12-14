# 빈 스코프
빈 스코프는 빈이 존재할 수 있는 범위를 말한다.

## 1. 스프링이 지원하는 스코프
- 싱글톤: 기본 스코프로 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프이다. (스코프를 생략하면 싱글톤 스코프이다.)
- 프로토타입: 스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입(+초기화 메서드)까지만 관여하고 더는 관리하지 않는 범위를 갖는다.
- 웹 관련 스코프
    + request
    + session
    + application

## 2. 스코프 등록
- 자동 등록: 스프링 빈으로 등록하고 싶은 클래스에 @Scope("~") 지정
- 수동 등록: 수동 등록하는 메서드에 @Scope("~") 지정

## 3. 프로토타입 스코프
스프링 컨테이너에 조회하면 스프링 컨테이너는 항상 새로운 인스턴스를 생성해서 반환한다.
- 실무에서 웹 애플리케이션을 개발해보면, 싱글톤 빈으로 대부분의 문제가 해결할 수 있기 때문에 프로토타입 빈을 직접적으로 사용하는 일은 드물다.

### 3.1 과정
1. 프로토타입 스코프의 빈을 스프링 컨테이너에 요청
2. 스프링 컨테이너는 이 시점에 프로토타입 빈을 생성하고, 필요한 의존 관계를 주입
3. 스프링 컨테이너는 생성한 프로토타입 빈을 클라이언트에 반환
4. 이 후 스프링 컨테이너에 같은 요청이 오면 항상 새로운 프로토타입 빈을 생성해서 반환
- 스프링 컨테이너는 이 후 생성된 프로토타입 빈을 관리하지 않으며 관리할 책임은 프로토타입 빈을 받은 클라이언트에게 있다. 
- 따라서 @PreDestory와 같은 종료 메서드는 실행되지 않으며 종료 하고싶을 때는 해당 프로토타입 빈 객체를 가져와 close()를 호출해야한다.

### 3.2 프로토타입 스코프와 싱글톤 빈을 함께 사용시 문제점
상황: 싱글톤 빈이 의존관계 주입을 통해 프로토타입 빈을 주입받아 사용 <br>
예상 결과: 싱글 톤 빈을 호출할 때마다 프로토타입 빈이 생성되야한다.
- 싱글톤 클래스의 logic 메서드를 통해 프로토타입의 addCount 메서드로 logic 메서드를 호출할 때마다 프로토타입 빈의 cnt가 1씩 증가하도록 구현
- 하지만 프로토타입 빈은 의존관계 주입을 통해 얻어진 1개의 빈 만이 존재하고 스프링 컨테이너는 관리하지 않는다.

### 3.3 Provider로 문제점 해결
해결: 싱글톤 빈이 프로토타입을 사용할 때 마다 스프링 컨테이너에 새로 요청하도록 구현하면된다.
- 이렇게 의존관계를 외부에서 주입(DI)하는 것이아닌 직접 필요한 의존관계를 찾는 것을 Dependency Lookup(DL) 의존관계 조회라고 한다.
- 하지만 이렇게 되면 해당 로직은 스프링 컨테이너에 종속적인 코드가 되고 단위 테스트도 어려워진다. 따라서 DL 정도의 기능만 제공하는 ObjectProvider를 사용한다.

### 3.4 ObjectProvider
지정한 빈을 컨테이너에서 대신 찾아주는 DL 서비스를 제공한다.

- ObjectProvider<DL하고싶은 빈 타입>는 스프링이 자동으로 빈으로 등록해주기 때문에 의존관계 주입으로 빈에서 불러오면된다.
- ObjectProvider의 getObject()를 호출하면 내부에서는 스프링 컨테이너를 통해 해당 빈을 찾아서 반환한다.
- 스프링이 제공하는 기능을 사용하지만(스프링 의존적), 기능이 단순하므로 단위 테스트를 만들거나 mock코드를 만들기는 훨씬 쉬워진다.
- 스프링이 지원하는 기능이기 때문에 별도의 라이브러리는 필요없다.

### 3.5 JSR-330 Provider
자바 표준으로 Provider를 사용하는 방법이다.
- javax.inject:javax.inject:1 라이브러리를 gradle에 추가해야한다.
- provider.get()을 호출할 때마다 프로토타입 빈을 요청한다.

### 3.6 ObjectProvider vs JSR-330 Provider
- ObjectProvider: DL을 위한 편의 기능을 많이 제공해주고 스프링 외에 별도의 의존관계 추가가 필요없기 때문에 편리하다. (스프링 사용시 적극 추천)
- JSR-330 Provider: 코드를 스프링이 아닌 다른 컨테이너에서도 사용할 수 있어야한다면 JSR-330 Provider을 사용해야한다.

## 4. 웹 스코프
웹 환경에서만 동작하며 스프링이 해당 스코프의 종료 시점까지 관리해주기 때문에 종료 메서드가 호출된다.

### 4.1 웹 스코프 종류
- request: HTTP 요청 하나가 들어오고 나갈 대 까지 유지되는 스코프, 각각의 HTTP 요청마다 별도의 빈 인스턴스가 생성되고, 관리된다.
- session: HTTP Session과 동일한 생명주기를 갖는 스코프
- application: 서블릿 컨텍스트와 동일한 생명주기를 갖는 스코프
- websocket: 웹 소켓과 동일한 생명주기를 갖는 스코프
- 모두 범위만 다를 뿐 동작 방식을 비슷하다.

### 4.2 request 스코프
상황: HTTP 요청이 오면 어떤 요청이 남긴 로그인지 구분하기 위해 request 스코프를 활용
- 기대하는 공통 포멧: [UUID][requsetURL]{message} (UUID로 구분)
- 해당 포멧의 구성을 갖는 MyLogger 클래스를 생성
- @Scope(value = "request")를 해당 클래스에 붙인다. 이제 빈이 HTTP 요청 당 하나씩 생성되고, 요청이 끝나는 시점에 소멸된다.
- @PostConstruct 초기화 메서드를 사용해서 uuid를 생성해서 저장해둔다.
- @PreDestory를 이용해서 종료 메시지를 남긴다.
- requestURL은 해당 빈이 생성되는 시점에는 알 수 없으므로, 외부에서 setter로 입력받는다.

### 4.3 스코프와 Provider
문제점: 웹 스코프는 웹 환경에서만 동작하기 때문에 request 스코프의 빈은 아직 생성되지 않는다. <br>
해결: Provider를 활용한다.
- ObjectProvider 덕분에 ObjectProvider.getObject()를 호출하는 시점까지 request 스코프 빈의 생성을 지연할 수 있다.
- ObjectProvider.getObject()를 호출하는 시점에 HTTP 요청이 진행 중이므로 request 스코프 빈의 생성이 정상 처리된다.
- MyLogger를 의존하는 다른 클래스들에 ObjectProvider.getObject()를 각각 한번씩 따로 호출해도 같은 HTTP 요청이면 같은 스프링 빈이 반환된다.

### 4.4 스코프와 프록시
MyLogger의 가짜 프록시 클래스를 만들어두고 HTTP request와 상관 없이 가짜 프록시 클래스를 다른 빈에 미리 주입해준다.
- @Scope(value = "request", proxyMode = ScopeProxyMode.TARGET_CLASS)를 추가
- 적용대상이 클래스면 TARGET_CLASS
- 적용대상이 인터페이스면 INTERFACES

### 과정
1. @Scope(value = "request", proxyMode = ScopeProxyMode.TARGET_CLASS)를 설정하면 스프링 컨테이너 CGLIB라는 바이트코드를 조작하는 라이브러리를 사용해서, MyLogger를 상속받는 가짜 프록시 객체를 생성한다.
2. myLogger라는 이름으로 해당 가짜 프록시 객체를 스프링 컨테이너에 등록한다.
3. 해당 가짜 프록시 객체를 의존관계 주입에 적용한다.
4. 실제 요청이 오면 실제 빈을 요청하는 위임 로직이 들어있어 그때 내부에서 진짜 빈을 요청한다.

### 프록시 객체
- 가짜 프록시 객체는 원본 클래스를 상속 받아 만들어졌기 때문에 해당 객체를 사용하는 클라이언트 입장에서는 원본인지 아닌지도 모르게 동일하게 사용가능하다. (다형성)
- 가짜 프록시 객체는 실제 request 스코프와는 관계 없이 단지 가짜일 뿐이고 싱글톤 처럼 동작한다. (싱글톤과 비슷하지만 다르게 동작하므로 주의해야한다.)
- 웹 스코프가 아니어도 사용가능하다.

### 이런 특별한 scope를 무분별하게 사용한다면 유지보수하기 어려워지므로 꼭 필요한 곳에만 최소화해서 사용하도록 하자.

## Reference
> 스프링 핵심 원리 - 기본편 [김영한]

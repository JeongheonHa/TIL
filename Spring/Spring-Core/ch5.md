## 스프링 컨테이너

### 1. 스프링 컨테이너 생성
```java
ApplicationContext ac = AnnotationConfigApplicationContext(AppConfig.class)
```
- AppConfig 설정 정보를 갖는 컨테이너를 생성한다는 뜻이다.
- ApplicationContext는 인터페이스, AnnotationConfigApplicationContext는 구현체이다.
- xml의 설정 정보를 갖는 컨테이너도 만들 수 있다.

### 2. 스프링 빈 등록
- 스프링 컨테이너는 파리미터로 넘어온 설정 클래스 정보를 사용해서 스프링 빈을 등록한다.
- key: 빈 이름, value: 빈 객체의 형식이며 빈 이름은 메서드 이름을 사용한다.
- 빈 이름은 항상 다른 이름을 부여해야한다.

### 3. 스프링 빈 의존 관계 설정을 준비한다.

### 4. 스프링 빈 의존 관계 설정을 완료한다.
- 스프링 컨테이너가 설정 정보를 참고해서 의존 관계를 주입한다.
- 스프링은 빈을 생성하고, 의존관계를 주입하는 단계가 나누어져있지만 자바 코드로 스프링 빈을 등록하면 생성자를 호출하면서 의존관계 주입도 한번에 처리한다.

### 5. 빈 출력하기

#### AnnotationConfigApplicationContext 클래스
- getBean(["빈 이름",] 빈 타입) : 해당 빈 이름과 빈 타입의 빈 객체를 반환
- getBeanDefinitionNames() : 스프링에 등록된 모든 빈의 이름을 반환
- getBeanDefinition("빈 이름") : 해당 빈에 대한 메타 데이터 정보

#### BeanDefinition 클래스
- getRole() : 스프링 내부에서 사용하는 빈을 구분
- ROLE_APPLICATION: 일반적으로 사용자가 정의한 빈
- ROLE_INFRASTRUCTURE: 스프링이 내부에서 사용하는 빈

```java
if (beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
    Object bean = ac.getBean(beanDefinitionName);
}
```

#### 조회 대상 스프링 빈이 없으면 예외가 발생한다.
- NoSuchBeanDefinitionException: No bean named 'xxxxx' available

#### 구체 타입으로 조회하면 변경시 유연성이 떨어진다.

#### 동일한 타입이 둘 이상인 경우의 조회
- 동일한 타입이 둘 이상인 경우 빈 이름을 지정해줘야한다.
- 빈 이름을 지정해주지 않을 경우 NoUniqueBeanDefinitionException 예외 발생

#### 특정 타입을 모두 조회
- ac.getBeansOfType(빈 타입): 해당 타입의 모든 빈을 Map<String, 빈 타입>으로 조회

### 6. 스프링 빈의 상속 관계
- 부모 타입으로 조회하면, 자식 타입도 함께 조회된다. (인터페이스를 구현한 클래스 관계도 포함)
- 부모 타입으로 조회시, 자식이 둘 이상 있으면, 빈 이름을 지정해준다.
    + 지정해 주지 않으면 NoUniqueBeanDefinitionException 예외 발생

### 7. BeanFactory
- BeanFactory(interface) <- ApplicationContext(interface) <- AnnotationConfigApplicationContext
- BeanFactory는 스프링 컨테이너의 최상위 인터페이스로 스프링 빈을 관리하고 조회하는 역할을 담당한다. (getBean...)
- BeanFacotry를 직접 사용할 일은 거의 없으며 부가 기능이 포함된 ApplicationContext를 사용한다.

### 8. ApplicationContext
- 애플리케이션을 개발할 때는 빈을 관리하고 조회하는 기능은 물론, 수 많은 부가 기능을 제공한다.
- BeanFactory나 ApplicationContext를 스프링 컨테이너라고 한다.

#### ApplicationContext의 부가 기능
모두 인터페이스
- MessageSource: 메시지 소스를 활용한 국제화 기능
    + 한국에서 들어오면 한국어로, 영어권에서 들어오면 영어로 출력해주는 기능

- EnvironmentCapable: 환경 변수
    + 로컬: 내 PC에서 개발하는 환경
    + 개발: test 환경
    + 운영: 실제 production이 나가는 환경
    + stage 환경: 운영과 거의 동일한 환경을 만드는 환경

- ApplicationEventPublisher: 애플리케이션 이벤트
    + 이벤트를 발행하고 구독하는 모델을 편리하게 지원
- ResourceLoader: 편리한 리소스 조회
    + 파일, 클래스패스, 외부 등에서 리소스를 추상화해서 편리하게 조회

### 9. xml 설정 정보를 이용해서 스프링 빈 등록
- XML을 사용하면 컴파일 없이 빈 설정 정보를 변경할 수 있지만 최근에는 스프링 부트를 많이 사용하면서 XML기반의 설정을 잘 사용하지 않는다.
- appConfig.xml과 AppConfig.java와 구성이 거의 비슷하다. (자세한건 실습한 코드를 보자.)

### 10. 스프링 컨테이너는 BeanDifinition인터페이스만 의존한다.
- BeanDifinition을 빈 설정 메타정보라 한다.
- @Bean 하나에 하나씩 메타 정보가 생성된다.
- 스프링 컨테이너는 이 메타 정보를 기반으로 스프링 빈을 생성한다.

#### 과정
AnnotationConfigApplicationContext는 AnnotationBeanDefinitionReader를 사용해서 AppConfig.class를 읽고 BeanDefinition을 생성하고 스프링 컨테이너(ApplicationContext)는 BeanDefinition에 의존한다.

#### BeanDefinition 정보
- BeanClassName: 생성할 빈의 클래스 명(자바 설정 처럼 팩토리 역할의 빈을 사용하면 없다)
- factoryBeanName: 팩토리 역할의 빈을 사용할 경우 이름 (appConfig)
- factorymemthodName: 빈을 생성할 팩토리 메서드 지정 (memberService)
- Scope: 싱글톤(기본값)
- lazyInit: 스프링 컨테이너를 생성할 때 빈을 생성하는 것이 아니라, 실제 빈을 사용할 때까지 최대한 생성을 지연처리 하는지 여부
- initMethodName: 빈을 생성하고, 의존관계를 적용한 뒤에 호출되는 초기화 메서드 명
- DestoryMethodName: 빈의 생명주기가 끝나서 제거하기 직전에 호출되는 메서드 명
- Constructor argument, Properties: 의존 관계 주입에서 사용한다. (자바 설정 처럼 팩토리 역할의 빈을 사용하면 없다)

## 싱글톤 컨테이너

### 1. 스프링이 없는 순수한 DI 컨테이너
스프링이 없는 순수한 DI 컨테이너는 같은 요청을 여러 번하면 같은 객체 여러개를 새로 생성함으로써 메모리 낭비가 발생한다.

### 2. 싱글톤 패턴
싱글톤 패턴을 이용해 해당 객체를 딱 1개만 생성되고, 해당 객체를 다른 요청들이 공유하도록 설계함으로써 해결할 수 있다.

```java
public class SingletonService {
    // 1. static 영역에 객체를 1개만 생성해 둔다.
    private static final SingletonService instance = new SingletonService();

    // 2. public으로 개방해서 객체 인스턴스가 필요하면 static 메서드를 통해서만 조회할 수 있도록 한다.
    public static SingletonService getInstance() {
        return instance;
    }

    // 3. 생성자를 private으로 감추어서 외부에서 해당 클래스의 인스턴스를 생성하지 못하도록 막는다.
    private SingletonService() {}

    public void logic() {
        System.out.println("싱글톤 객체 로직 호출")
    }
}
```

- 싱글톤 패턴을 구현하는 방법은 여러가지가 있으며 해당 방법은 객체를 미리 생성해두는 가장 단순하고 안전한 방법이다.
- 스프링 컨테이너는 자체적으로 기본값이 싱글톤 패턴이기 때문에 AppConfig에서 객체를 생성해서 의존 관계를 주입했던 것을 getInstance()로 바꿀 필요가 없다.

### 3. 싱글톤 패턴의 문제점
- 코드 자체가 길어진다.
- 클라이언트가 구체 클래스에 의존하게 된다. (DIP 위반, 구체 클래스.getInstance())
- 클라이언트가 구체 클래스에 의존해서 OCP원칙을 위반할 가능성이 높다.
- 싱글톤은 클래스 내부에 객체를 하나 생성해 놓기 때문에 유연한 test가 어렵다.
- 내부 속성을 변경하거나 초기화 하기 어렵다.
- private 생성자로 자식 클래스를 만들기 어렵다.
- 유연성이 떨어지며 안티패턴으로 불리기도 한다.

### 4. 싱글톤 문제점 해결
- 스프링 프레임워크는 싱글톤의 모든 문제점을 해결하고 객체는 싱글톤 패턴으로 관리해준다.
- 스프링 컨테이너는 싱글톤 패턴을 적용하지 않아도 객체 인스턴스를 싱글톤으로 관리한다.
- 이렇게 싱글톤 객체를 생성하고 관리하는 기능을 싱글톤 레지스트리라고 한다.

### 5. 싱글톤 방식의 주의점
- 객체 인스턴스를 하나만 생성해서 공유하는 모든 싱글톤 방식은 여러 클라이언트가 하나의 동일한 객체를 공유하기 때문에 상태 유지(stateful)하게 설계하면 안된다.
- 무상태로 설계해야한다.
    + 특정 클라이언트에 의존적인 필드가 있으면 안된다.
    + 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다.
    + 가급적 읽기만 가능해야한다.
    + 필드 대신 공유되지 않는 지역변수, 파라미터, ThreadLocal 등을 사용해야한다.

### 6. @Configuration과 싱글톤
- memberService 빈을 만드는 코드에서 memberRepository()를 호출하면 의존 관계의 객체를 생성한다.
- orderService 빈을 만드는 코트에서 memberRepository()를 호출하면 의존 관계의 객체를 생성한다.
- 두 개의 빈을 만드는 과정에서 memberRepository()를 2번 호출해 의존 관계의 동일한 객체 2개가 생성되어 싱글톤이 깨진 것 처럼 보이지만 @Configuration이 해결해준다.

#### @Configuration과 바이트 코드 조작
- @Configuration은 내가 만든 클래스가 아닌 스프링이 CGLIB 라는 바이트코드 조작 라이브러리를 사용해 AppConfig 클래스를 상속받는 임의의 다른 클래스를 만들고, 그 다른 클래스를 스프링 빈으로 등록한 것이다.
- 또한 AppConfig 설정 정보를 이용한 컨테이너를 생성할 때 AppConfig 클래스 또한 빈으로 저장된다.
- 이때, CGLIB 라는 바이트코드 조작 라이브러리로 인해 AppConfig@CGLIB이 빈으로 저장되고 getBean(AppConfig.class)로 꺼내도 조상 타입으로 자손 타입을 꺼내는 것이기 때문에 자손 타입인 AppConfig@CGLIB 인스턴스가 반환된다. 
- 또한, 바이트코드 조작에 의해 @Bean이 붙은 메서드 마다 스프링 컨테이너에 해당 빈이 존재하면 빈을 반환하고 해당 빈이 없으면 생성해서 빈으로 등록하고 반환하는 코드가 동적으로 만들어진다.

#### @Configuration 없이 @Bean만 있는 경우
- CGLIB 기술 없이 순수 AppConfig로 스프링 빈에 등록되고 memberRepository()호출도 여러 번 호출됨으로서 싱글톤이 깨진다.
- 또한 이렇게 의존관계를 위해 메서드를 호출해 생성된 구현 객체들은 스프링 빈에 등록되지 않고 스프링의 관리를 받지않는 객체가 된다는 사실을 알 수 있다.

#### 스프링 설정 정보는 항상 @Configuration을 사용하도록 하자.

## Reference
> 스프링 핵심 원리 - 기본편 [김영한]

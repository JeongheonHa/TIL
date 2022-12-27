# 컴포넌트 스캔
스프링 설정 정보 없이 자동으로 스프링 빈을 등록하는 기능을 제공

## 1. @ComponentScan
AppConfig 클래스에 @ComponentScan 애너테이션을 붙임으로써 @Component가 붙은 클래스들을 스프링 빈으로 자동 등록한다. 또한 @Configuration도 @Component 애너테이션이 붙어있기 때문에 스캔의 대상이 된다.

- @ComponentScan은 @Component가 붙은 모든 클래스를 스프링 빈으로 등록한다.
- 등록 할 때 이름은 클래스 명에서 앞 글자만 소문자로 바꾼 이름을 등록한다.
- @Component("이름")을 통해 스프링 빈 이름을 직접 정의할 수 도 있지만 왠만하면 자동 등록된 이름을 사용하자.

## 2. @Autowired
생성자에 @Autowired를 붙여줌으로써 여러 의존 관계를 자동으로 주입해준다.

- 스프링 컨테이너가 해당 타입으로 스프링 빈을 찾아서 주입한다.
- 타입이 일치한 빈이 여러 개 있을 경우

## 3. 탐색 위치와 기본 스캔 대상
모든 자바 클래스를 컴포넌트 스캔하면 시간이 굉장히 오래 걸리기 때문에 탐색 위치를 지정할 수 있다.

```java
@ComponentScan(
    basePackages = "hello.core",
)
```

- basePackages는 탐색할 패키지의 위치를 지정하고 해당 패키지를 포함한 모든 하위 패키지들을 탐색한다.
- basePackageClasses: 지정한 클래스의 패키지를 탐색 시작 위치로 지정한다.
- 만일 탐색 위치를 지정하지 않는다면 @ComponentScan이 붙은 설정 정보 클래스의 패키지가 시작 위치가 되기 때문에 탐색 위치를 생략하고 설정 정보 클래스를 프로젝트 최상단에 두는 것을 추천한다.
- 스프링 부트를 사용하면 스프링 부트의 대표 시작 정보인 @SpringBootApplication을 프로젝트 시작 루트 위치에 두는 것이 관례이며 해당 설정안에 @ComponentScan이 들어있다.

### 컴포넌트 스캔 기본 대상
- @Component: 컴포넌트 스캔에서 사용
- @Controller: 스프링 MVC컨트롤러에서 사용
- @Service: 스프링 비즈니스 로직에서 사용
- @Repository: 스프링 데이터 접근 계층에서 사용 + 데이터 계층의 예외를 스프링 예외로 변환
- @Configuration: 스프링 설정 정보에서 사용 + 스프링 빈이 싱글톤을 유지하도록 추가 처리

### 애너테이션은 상속관계라는 것이 없다.
애너테이션이 특정 애너테이션을 들고있는 것을 인식할 수 있는 것은 자바 언어가 아닌, 스프링이 지원하는 기능이다.

### 필터
- includeFilters: 컴포넌트 스캔 대상을 추가로 지정
- excludeFilters: 컴포넌트 스캔에서 제외할 대상을 지정
- @Component면 충분하기 때문에 includeFilters는 사용할 일이 거의 없고 excludeFilters는 여러가지 이유로 가끔 사용한다.
```java
// MyIncludeComponent 애너테이션이 붙은 클래스는 스프링 빈에 등록
// MyExcludeComponent 애너테이션이 붙은 클래스는 스프링 빈에서 제거
@ComponentScan(
    includeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyIncludeComponent.class),
    excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyExcludeComponent.class)
)
static class ComponentFilterAppConfig {}
```
- ANNOTATION: 기본값, 애너테이션을 인식해서 동작 (자주 사용)
- ASSIGNABLE_TYPE: 저정한 타입과 자식 타입을 인식해서 동작 (가끔 사용)
- ASPECTJ AspectJ패턴 사용 (거의 사용x)
- REGEX: 정규 표현식 (거의 사용x)
- CUSTOM: TypeFilter라는 인터페이스를 구현해서 처리 (거의 사용x)

## 4. 컴포넌트 스캔에서 같은 빈 이름을 등록하는 경우

### 자동 빈 등록 vs 자동 빈 등록
컴포넌트 스캔에 의해 자동으로 스프링 빈이 등록될 때 이름이 동일한 경우 스프링은 ConfictingBeanDefinitionException 예외가 발생한다. 따라서 충돌하는 경우는 발생하지 않는다.

### 수동 빈 등록 vs 자동 빈 등록
수동 빈이 자동 빈을 오버라이딩 함으로써 수동 빈 등록이 우선권을 갖는다.
- 하지만, 이렇게 만든 경우 잡기 어려운 애매한 버그가 발생할 가능성이 크기 때문에 애초에 동일한 이름의 빈을 등록하는 상황을 만들면 안된다.
- 스프링 부트에서는 수동 빈과 자동 빈 등록이 충돌이 나면 오버라이딩 하지 못하게 오류가 발생하는 것이 기본값이다. (물론 true로 변경 가능하다.)

## Reference
> 스프링 핵심 원리 - 기본편 [김영한]

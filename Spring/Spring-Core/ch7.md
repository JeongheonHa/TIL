# 의존 관계 자동 주입

## 1. 의존 관계 주입 방법 4가지
- 생성자 주입
- 수정자 주입
- 필드 주입
- 일반 메서드 주입

## 2. 생성자 주입
생성자를 통해서 의존 관계를 주입 받는 방법
- 생성자 호출 시점에 딱 1번만 호출되는 것이 보장된다.
- 값을 세팅하고 더 이상 변경하지 않을 때 사용(불변)
- 외부에서 해당 클래스를 사용할 때 생성자에 있는 것을 모두 다 넣어줘야한다. (필수)
- 생성자 주입은 final을 붙이기 때문에 필드의 모든 참조변수에 객체를 생성해서 넣어줘야한다. + 정확히 1개만 생성하도록 제한한다.

```java
@Component
public class OrderServiceImpl implements OrderService {

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    @Autowired // 생성자가 1개일 경우 @Autowired 생략 가능
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```

### 생성자 주입에서의 등록과 의존관계 주입
스프링 빈을 등록할 때는 원래 등록과 의존 관계 주입 두 단계로 나누어져 있지만 생성자 주입의 경우 해당 클래스를 등록하는 과정에서 객체를 생성하기 위해 생성자를 호출해 파리미터로 의존 관계의 객체를 가져와서 객체를 생성해야하기 때문에 등록과 의존관계 주입이 동시에 이루어진다. 또한 의존 관계의 빈을 가져와야할 때 등록되어 있지 않은 경우 등록 후 가져온다.

### 생성자를 사용해야하는 이유
- 불변: 대부분의 의존관계 주입은 한번 일어나면 애플리케이션의 종료시점까지 의존관계를 변경할 일이 없고 변경해서도 안된다.
- 누락: 프레임워크 없이 순수한 자바 코드를 단위 테스트 하는 경우가 굉장히 많은데 생성자 주입으로 작성하면 객체를 생성할 때 의존관계의 객체들을 인자로 넣어주지 않으면 컴파일 에러를 발생시키기 때문에 한번에 의존관계를 파악할 수 있지만 다른 방식들은 의존관계를 한번에 알아볼 수 없다. 따라서 프레임워크에 의존하지 않는 순수한 자바 언어의 특징을 살릴 수 있다.

- final: 생성자에서 혹시라도 값이 설정되지 않는 오류를 컴파일 시점에 막아준다.
    + 생성자 주입만 final 사용 가능: 수정자 주입의 경우 객체가 생성된 다음 참조 변수 값을 변경하는 것이기 때문에 final 사용 불가

## 3. 수정자 주입(setter 주입)
필드의 값을 변경하는 수정자 메서드를 통해서 의존관계를 주입하는 방법
- 선택, 변경 가능성이 있는 의존관계에 사용
- 선택: final이 없기 때문에 해당 객체를 생성해도 되고 안해도된다.
- 변경: 중간에 의존관계의 객체를 바꾸고 싶다면 수정자를 호출하면 된다.

```java
@Component
public class OrderServiceImpl implements OrderService {
    private MemberRepository memberRepository;
    private DiscountPolicy discountPolicy;

    @Autowired
    public void setMemberRepository(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    @Autowired
    public void setDiscountPolicy(DiscountPolicy discountPolicy) {
        this.discountPolicy = discountPolicy;
    }
}
```
- 수정자 주입의 경우 memberRepository가 빈으로 등록되어 있지 않아도 의존관계 주입을 안하면 되므로 사용이 가능하다 (등록, 의존관계 주입 단계가 나눠지므로)
- @Autowired의 기본 동작은 주입할 대상이 없으면 오류가 발생하기 때문에 주입할 대상이 없어도 동작하게 하기 위해선 @Autowired(required = false)로 지정해야한다.

## 4. 필드 주입
필드에 바로 주입하는 방법 (사용하지 말자)
- 애플리케이션의 실제 코드와 관계 없는 스프링 컨테이너에서의 테스트 코드 (애플리케이션과 관련된 코드에서는 사용x)

```java
@Component
public class OrderServiceImpl implements OrderService {

    @Autowired
    private MemberRepository memberRepository;

    @Autowired
    private DiscountPolicy discountPolicy;
}
```

### 장점
- 코드가 간결하다.

### 단점
- DI 프레임워크가 없으면 아무것도 할 수 없다. <br>
스프링 빈에서 필드에 있는 타입의 객체를 가져오는 것일 뿐이기 때문에 스프링 빈을 사용하지 않는 test 같은 경우 개별 객체를 생성해야하지만 객체를 생성할 방법이 없기 때문에 NullPointerException이 발생할 것이다. (수정자 주입으로 가져올 수 있지만 그럴거면 처음부터 수정자 주입을 쓰는게 낫다.)

## 5. 일반 메서드 주입
일반 메서드를 통해서 주입 받는 방법
- 한번에 여러 필드를 주입 받을 수 있다.
- 일반적으로 생성자 주입과 수정자 주입에서 모두 해결할 수 있기 때문에 사용하지 않는다.

```java
@Component
public class OrderServiceImpl implements OrderService {
    private MemberRepository memberRepository;
    private DiscountPolicy discountPolicy;

    @Autowired
    public void init(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```

## 6. 옵션 처리
주입할 스프링 빈이 없어도 동작해야 할 경우 자동 주입 대상을 옵션으로 처리하는 3가지 방법

### @Autowired(required=false)
자동 주입할 대상이 없으면 수정자 메서드 자체가 호출되지 않는다.

### org.springframwork.lang.@Nullable
자동 주입할 대상이 없으면 null이 입력된다.

### Optional<>
자동 주입할 대상이 없으면 Optional.empty가 입력된다.

### @Nullable, Optional은 스프링 전반에 걸쳐서 지원된다.

## 7. 롬복

### @Getter, @Setter, @ToString
- @Getter, @Setter: getter, setter 메서드를 자동으로 생성해준다.
- @Tostring: 객체의 내용을 toString으로 오버라이딩한 것을 자동으로 생성해준다.

### @RequiredArgsConstructor
- final이 붙은 필드를 모아서 생성자를 컴파일 시점에 자동으로 생성해준다.

### 최신 트렌드
최근에는 생성자를 1개만 두고 @Autowired를 생략하는 방법을 주로 사용하며 여기에 @RequiredArgsConstructor와 함께 사용하여 코드를 간결하게 해준다.

## 8. @Autowired로 의존관계를 주입할 때 조회 빈이 2개 이상인 경우
@Autowired는 빈의 타입으로 조회하기 때문에 같은 타입의 빈이 2개 이상인 경우 NoUniqueBeanDefinitionException 예외가 발생한다.

### @Autowired 필드 명
@Autowired는 1차로 타입 매칭을 시도하고, 이때 같은 타입의 빈이 여러개 있을 경우 2차로 필드 이름, 파리미터 이름으로 빈 이름을 매칭한다.
- 따라서, 필드 명이나 파리미터 명을 해당 빈의 이름으로 설정함으로서 해결할 수 있다.

### @Qualifier
스프링 빈에 등록할 같은 타입의 구현체들에 @Qualifier("이름")애너테이션을 붙이고 생성자나, 수정자의 의존관계의 파라미터 앞에 @Qualifier("이름")애너테이션을 붙여 지정해준다. (해당 이름으로 빈에 저장된다.)

- Qualifier로 주입할 때 해당 이름의 구현체를 찾지 못한 경우 해당 이름의 스프링 빈을 추가로 찾고 그래도 없으면 NoSuchBeanDefinitionException 예외를 발생시킨다.
- 하지만, @Qualifier는 @Qualifier를 찾는 용도로만 사용하는 것이 좋다.

### 애너테이션으로 만들어 사용하기
@Qualifier("이름")을 사용하면 이름은 문자열이기 때문에 컴파일시 타입 체크가 안되기 때문에 따로 애너테이션을 만들어 사용하면 코드를 더 깔끔하게 유지 할 수 있다.

```java
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@Qualifier("mainDiscountPolicy")
public @interface MainDiscountPolicy {
}
```

### @Primary
@Autowired시 같은 타입의 여려 빈이 매칭되면 @Primary가 붙은 구현체가 우선순위를 갖는다.

### @Primary vs @Qualifier
- 스프링은 자동보다는 수동, 넓은 범위보다는 좁은 범위은 구체적인 것이 우선순위가 높기 때문에 @Qualifier가 우선순위가 높다.
- @Primary: 메인 데이터베이스의 커넥션을 획득하는 스프링 빈은 @Primary을 적용해서 편리하게 조회하는 것이 유리하다.
- @Qualifier: 서브 데이터베이스 커넥션 빈을 획득할 때 @Qualifier를 적용해서 명시적으로 얻는 방식으로 하는 것이 코드를 더 깔끔하게 유지 시켜준다.

### 동일한 타입의 조회한 빈이 모두 필요한 경우 (List, Map)
상황: DiscountService 클래스에서 동일한 타입의 조회한 빈이 모두 필요하다고 할 때
- 동일한 타입의 의존관계에 있는 구현체들을 의존관계 주입으로 해당 객체 내에 Map이나 List로 모두 저장한다. (list는 이름으로만 저장된다.) 
- 이 후 해당 클래스 내에 파라미터로 빈 이름을 받아 Map에서 해당 빈 이름(key)에 맞는 객체(value)를 꺼내는 로직을 만들어 원하는 구현체의 메서드나 변수를 사용할 수 있도록 구현한다.
- 클라이언트 입장에서는 DiscountSerivce의 로직에 인자로 사용하고자 하는 구현체의 이름을 넘겨줌으로써 자신의 입 맛에 맞게 구현체를 선택해서 사용할 수 있다.

### 스프링 컨테이너를 생성할 때 파라미터로 여러 개의 클래스를 스프링 빈으로 등록할 수 있다.
```java
ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class, DiscountService.class);
```
이렇게 하면 해당 클래스들을 동시에 스프링 빈으로 자동 등록한다.

## 9. 언제 자동 / 수동을 사용해야할까

### 편리한 자동 기능을 기본으로 사용하자 (자동 빈 등록)
설정 정보를 기반으로 애플리케이션을 구성하는 부분과 실제 동작하는 부분을 명확하게 나누는 것이 이상적이지만 @Component 하나만 넣어주면 끝나는 일을 부가적인 처리가 많아진다. 또한 스프링은 시간이 갈수록 자동을 선호해가는 추세이며 무엇보다 자동 빈 등록을 사용해도 애너테이션을 조금 수정하는 부분을 제외하면 OCP, DIP를 지킬 수 있다.
- 업무 로직 빈: 컨트롤러, 서비스, 리포지토리 등
- 업무 로직은 문제가 발생해도 어떤 곳에서 발생했는지 명확히 파악하기 쉽기 때문에 자동 빈으로

### 수동 빈 등록의 사용
애플리케이션에 광범위하게 영향을 미치는 기술 지원 객체는 수동 빈으로 등록해서 설정 정보에 바로 나타나게 하는 것이 유지보수에 좋다.
- 기술 지원 빈: 기술적인 문제나 공통 관심사(AOP)를 처리할 때 주로 사용 (데이터베이스 연결, 공통 로그 처리와 같은 업무 로직을 지원하기 위한 하부 기술이나 공통 기술들)
- 기술 지원 로직은 애플리케이션 전반에 걸쳐 광범위하게 영향을 미쳐 적용이 잘 되고 있는지 아닌지 조차 파악하기 어렵기 때문에 수동 빈 등록을 사용하여 명확하게 드러내는 것이 좋다.
- 설정 정보만 봐도 한눈에 빈의 이름과 어떤 빈이 주입될지 파악하기 위해서 다형성을 적극 활용하는 비즈니스 로직의 경우 수동 등록을 고민해봐야한다.

## Reference
> 스프링 핵심 원리 - 기본편 [김영한]

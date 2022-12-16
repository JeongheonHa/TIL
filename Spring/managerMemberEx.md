# 회원 관리 예제

목표 : 회원 관리 예제를 통해 Spring의 전반적인 내용을 알아봅시다. <br>

## 일반적인 웹 애플리케이션의 계층 구조
컨트롤러, 서비스, 리포지토리, 도메인의 형태를 갖는다.<br>
- 컨트롤러 : 웹 MVC의 컨트롤러 역할을 수행
- 서비스 : 회원의 중복 가입이 안된다는 규칙과 같은 비즈니스 로직을 구현
- 리포지토리 : DB 접근 및 도메인 객체를 DB에 저장하고 관리, 핵심 비즈니스 로직이 잘 동작 할 수 있게 구현한 객체
- 도메인 : 비즈니스 도메인 객체 

## 클래스 의존 관계
아직 데이터 저장소가 결정되지 않았기 때문에 interface로 구현 후 나중에 바꿔 주도록 설계

<img src = "https://user-images.githubusercontent.com/108064146/208070482-75d8cf82-7f17-4f33-bace-e785684d8b6c.png">

## domain
- 아이디와 이름을 갖는 객체를 생성

## repository

### MemberRepository 인터페이스 <br>
<img src="https://user-images.githubusercontent.com/108064146/208071530-f475deb5-2c2f-4b59-9080-dfb68b76f552.png">

- Optional<>은 null이 발생할 가능성이 있는 객체를 감싸 여러 메서드를 사용할 수 있게해준다.

### MemoryMemberRepository <br>
<img src="https://user-images.githubusercontent.com/108064146/208072481-dbfd31a1-3cfc-4f89-a20e-f88ddcf11382.png">

- 실무에서는 동기화 문제 때문에 HashMap보다는 ConcurrentHashMap을 사용한다.
- 기능이 잘 동작할 수 있게 적절히 오버라이딩 해준다.

## Test
<img src="https://user-images.githubusercontent.com/108064146/208073152-283315a6-b092-4229-9530-3c00dbae4e96.png">

- test는 순서를 보장하지 않기 때문에 모든 테스트는 메서드의 순서와 상관없이 따로 동작 할 수 있도록 설계해야한다. (main메서드를 작성하듯)
- 따라서, test메서드가 끝날 때마다 반복적으로 메서드를 호출하는 AfterEach 애너테이션을 이용해 공용 데이터를 모두 clear 해준다.
- 두 객체를 비교할 때는 `Assertions.assertThat(객체).isEqualTo(객체)`의 형식을 사용하도록 하자.


## service

### 의존성 부여 (Dependency injection)
<img src="https://user-images.githubusercontent.com/108064146/208074680-05c40dd4-1290-4e82-a9ee-5d03b9e77c95.png">

- 리포지토리와 서비스간의 의존성을 부여한다.

### 로직
<img src="https://user-images.githubusercontent.com/108064146/208075165-d3033e02-4d63-46c4-8e2c-ef18a30ae895.png">

- 회원 가입과 회원 조회와 같은 비즈니스 로직을 구현한다.

### Test
<img src="https://user-images.githubusercontent.com/108064146/208075894-46d93a02-1997-47ae-beca-615861f8aff3.png">

- BeforeEach 애너테이션은 테스트 전에 호출된다. 테스트가 서로 영향을 주지 않도록 새로운 객체를 생성하여 의존 관계를 맺어준어야한다.
- test를 작성할 때는 given / when / then의 순서에 맞게 작성하는 것이 좋다.

<img src="https://user-images.githubusercontent.com/108064146/208076497-f32fc250-1ba7-41ad-a3d0-832ea8535960.png">

- test를 작성할 때는 예외가 발생할 가능성이 있으면 예외에 대한 test cass도 작성해야한다.
- 해당 메서드는 예외가 발생하지 않았거나 다른 예외가 발생될 경우 실패한다. try-catch보다 해당 메서드를 쓰도록 하자.

## controller

<img src="https://user-images.githubusercontent.com/108064146/208082593-28ff45a5-df50-409f-bf32-a306d1425d1d.png">

- @GetMapping("url") 해당 url로의 진입이 시도가 되면 아래의 메서드를 실행하다. (보통 조회에 쓰인다.)
- return "~"은 templates의 return 값과 같은 이름의 view를 반환 해준다.
- 이 후 viewresolver가 return 문자열에 해당하는 templates 폴더를 연결한 후 thymeleaf 템플릿 엔진에 처리해달라고 넘기면 엔진에서 렌더링 후 변환한 html을 웹 브라우저에 반환한다.

- @PostMapping은 주어진 url과 일치하는 http post요쳥을 처리한다. (보통 등록할 때 사용된다.)
- form 형태의 객체에 html에서 입력 받은 이름을 form 객체에 저장하여 파라미터로 받아온다.
- return "redirect:/"는 다시 홈으로 돌려보낸다는 의미이다.

### DI
<img src="https://user-images.githubusercontent.com/108064146/208078245-69b751dd-0403-4e29-bce8-0df23bab7175.png">
- controller와 service간의 의존성을 부여해준다.

## Dependency Injection (의존성 주입)
객체 의존관계를 외부에서 넣어주는 것
- 의존성을 부여하는 방법에는 component방식과 config방식이 있다.
- DI에는 필드 주입, setter 주입, 생성자 주입이 있으며 이 중 생성자 주입을 사용하도록 하자.


### component 방식
- @Controller, @Service, @Repository 모두 @Component 방식이며 해당 애너테이션이 있으면 스프링이 뜰 때 해당 객체를 생성하여 스피링 컨테이너dml 스프링 빈에 등록한다. @Autowired가 해당 클래스의 생성자에 있으면 연관된 객체를 스프링 컨테이너에서 찾아서 넣어준다. 
- 스프링은 스프링 빈에 객체를 저장할 때 싱글 톤 방식으로 등록한다.

### config 방식
자바 코드로 직접 스프링 빈에 등록하는 방식
- 서비스, 리포지토리의 Component와 Autowired애너테이션을 제거한 상태에서 hellospring패키지 아래에 SpringConfig클래스에 작성한다.

<img src="https://user-images.githubusercontent.com/108064146/208080547-d4edcdcb-72d6-44ac-8ff7-16ac3e401eb3.png">
- DI 메서드를 한 곳에 몰아 놓아 나중에 리포지토리를 변경하기 유용하게 할 수 있다.

### component vs config
config :  외부 라이브러리 또는 내장 클래스를 bean으로 등록하고자 할 경우 사용 <br>
component : 개발자가 직접 작성한 클래스를 bean으로 등록하고자 할 경우 사용
<br>
<br>

## Reference
스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술 [김영한]

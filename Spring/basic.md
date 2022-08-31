# 라이브러리

## 1. 스프링 부트 라이브러리

- spring-boot-starter-web
    + spring-boot-starter-tomcat : 톰캣(웹서버)
    + spring-webmvc : 스프링 웹 MVC

- spring-boot-starter-thymeleaf : 타임리프 템플릿 엔진(View)
- spring-boot-starter(공통) : 스프링 부트 + 스프링 코어 + 로깅
    + spring-boot
        - spring-core
    + spring-boot-starter-logging
        - logback,slf4j 두 가지 조합을 많이 사용

## 2. 테스트 라이브러리
- spring-boot-starter-test
    + junit : 테스트 프레임워크 (기본)
    + Mockito : 목 라이브러리
    + assert : 테스트 코드를 좀 더 편하게 작성하게 도와주는 라이브러리
    + spring-test : 스프링 통합 테스트 지원

# View 환경설정

## 1. Welcome Page 만들기
- resources/static/index.html을 만들어 Welcome Page 제작
```html
<!DOCTYPE HTML>
<html>
<head>
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
Hello
<a href="/>hello">hello</a>
</body>
</html>
```

- 스프링 부트가 제공하는 Welcome Page 기능
    + 웹 브라우저에 해당 파일을 넘겨주는 것뿐이다. (정적 페이지)

- thymeleaf 템플릿 엔진
    + 동작하고 프로그래밍되는 페이지를 만들 수 있다.

- controller
    + java/hello/hellospring/controller 패키지에 controller 클래스 생성
    + 웹 어플리케이션에서 /hello라고 들어오면 @GetMapping("hello")가 붙은 메서드를 호출

```java
// controller class
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HelloController {
    @GetMapping("hello")
    public String hello(Model model) {
        model.addAttribute("data", "hello!!");
        return "hello";
    }
}
```

- template
```html
<!DOCTYPE HTML>
<html xmlns:th="http:/>/www.thymeleaf.org"> // templete엔진으로서 thymeleaf 문법을 사용할 수 있도록 선언해준다.
<head>
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
<p th:text="'안녕하세요 ' + ${data}" >안녕하세요. 손님</p>
</body>
</html>
```

- controller 클래스의 hello 값 -> template의 data (키에 매핑되어있는 값을 template의 data에 치환)

- 과정
    + 웹 브라우저에 localhost:8080/hello 라고 던진다.
    + 스프링 부트가 내장하고 있는 내장 톰켓 서버에 받아 스프링에게 던진다.
    + 스프링의 hellocontroller 클래스에서 hello URL에 매칭 (@GetMapping("hello"))
    + controller에 있는 @GetMapping("hello")가 붙은 메서드가 실행 (return hello)
    + model에 key, value를 templates의 hello.html파일에 렌더링하도록 넘겨준다.
    + 스프링 부트가 templates/hello.html파일을 찾아서 Thymeleaf템플릿 엔진이 처리해준다.

- 컨트롤러에서 리턴 값으로 문자를 반환하면 `viewResolver`가 화면을 찾아서 처리한다.
    + 스프링 부트 템플릿엔진 기본 viewName매핑
    + `resources:templates/`+{ViewName}+`.html`

## 2. 빌드하고 실행하기
1. `./gradlew build`
2. `cd build/libs`
3. `java -jar hello-spring-0.0.1-SNAPSHOT.jar`
4. 실행 확인

- `./gradlew clean build` : 빌드 삭제

# 스프링 웹 개발 기초

## 1. 정적 컨텐츠
- 파일을 웹 브라우저에 그대로 내려주는 방식
- spring boot가 자동으로 제공

## 2. MVC와 템플릿 엔진
- MVC : `Model`, `View`, `Controller`
- html을 서버에서 프로그래밍해서 동적으로 바꿔서 내리는 방식
- `View` : 화면과 관련된 일만 작성 
- `Controller` : 비지니스 로직이나 서버 뒷단에 관련된 일 등
- `Model` : 화면에 필요한 것들을 담아서 화면에 넘겨주는 역할
- `viewResolver` : 뷰를 찾아서 템플릿 엔진을 연결시켜준다
```java
// controller
@GetMapping("hello-mvc")
    public String helloMvc(@RequestParam("name") String name, Model model) {
        model.addAttribute("name", name);
        return "hello-template";
    }
```
```html
// template
<html xmlns:th="http://www,thymeleaf.org">
<body>
<p th:text="'hello ' + ${name}">hello! empty</p>
</body>
</html>
```
- 실행 : http://localhost:8080/hello-mvc?name=spring
- 과정
    + 웹 브라우저가 localhost:8080/hello-mvc를 내장 톰켓 서버에 넘겨준다.
    + 내장 톰켓 서버는 스프링 부트에 넘겨준다.
    + 스프링 부트의 controller에서 매핑되어있는 메서드를 실행 return hello-template, model(name:spring)을 viewResolver에 넘겨준다
    + viewResolver에서 return과 같은 이름의 템플릿을 찾아서 Thymeleaf템플릿 엔진에게 처리해달라고 넘긴다. (templates/hello-template.html)
    + 템플릿 엔진이 렌더링을 통해 변환한 HTML을 웹 브라우저에 반환 

## 3. API
- json이라는 데이터 구조 포맷으로 client에게 데이터를 전달하는 방식 
```java
// controller
// 문자를 전달할 경우
@GetMapping("hello-string")
    @ResponseBody   // http body부에 데이터("hello " + name)를 직접 넣어 주겠다는 의미
    public String helloString(@RequestParam("name") String name) {
        return "hello " + name;
    }
// 문자의 경우 그냥 http에 반환
```
```java
// 객체를 전달할 경우
@GetMapping("hello-api")
    @ResponseBody
    public Hello helloApi(@RequestParam("name") String name) {
        Hello hello = new Hello();
        hello.setName(name);
        return hello;
    }

    static class Hello {
        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }
    // cmd + N : generate 단축키
```
- 실행 : http://localhost:8080/hello-api?name=spring
- 과정
    + 웹 브라우저가 localhost:8080/hello-api를 내장 톰켓 서버에 넘겨준다.
    + 내장 톰켓 서버는 스프링 부트에 넘겨준다.
    + controller의 @GetMapping으로 이동
    + @ResponseBody가 선언되어있기 때문에 객체(return: hello(name:spring))을 넘겨주면서 HttpMessageConverter가 작동
    + return 값이 String이면 StringConverter 동작, 객체면 JsonConverter 동작
    + 객체를 Json 스타일로 바꿔서 웹 브라우저에게 반환
    
- `@ResponseBody`사용 원리
    + HTTP의 BODY에 문자 내용을 직접 반환
    + `viewResolver` 대신 `HttpMessageConverter`가 동작
    + 기본 문자처리 : `StringHttpMessageConverter`
    + 기본 객체처리 : `MappingJackson2HttpMessageConverter`
    + byte처리 등 기타 여러 HttpMessageConverter가 기본으로 등록되어 있음

# Reference
> 스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB접근 기술 [김영한]

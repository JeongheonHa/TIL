# 스프링 MVC 구조 이해

<img src = "https://user-images.githubusercontent.com/108064146/210412150-76b89f98-fb06-40a9-985e-63c849b87454.jpeg">

## 1. 직접 만든 프레임워크와 비교
- FrontController -> DispatcherServlet
- handlerMappingMap -> HandlerMapping
- MyHandlerAdapter -> HandlerAdapter
- ModelView -> ModelAndView
- viewResolver -> ViewResolver
- MyView -> View

## 2. DispatcherServlet
DispatcherServlet도 부모 클래스에서 HttpServlet을 상속 받아서 사용하고, 서블릿으로 동작한다.
- DispatcherServlet -> FrameworkServlet -> HttpServletBean -> HttpServlet
- 스프링 부트는 DispatcherServlet을 서블릿으로 자동으로 등록하면서 모든 경로에 대해서 매핑한다.

### 2.1 요청 흐름
<img src = "https://user-images.githubusercontent.com/108064146/210416521-21749832-fc4a-4532-ad43-4ee79cc7f4b8.png">

1. 서블릿이 호출되면 HttpServlet이 제공하는 service()가 호출된다.
2. 스프링 MVC는 DispatcherServlet의 부모인 FrameworkServlet에서 service()를 오버라이딩 해두었다.
3. FrameworkServlet.service()를 시작으로 여러 메서드가 호출되면서 최종적으로 DispatcherServlet.`doDispatch()`가 실행된다.

### 2.2 doDispatch()
FrontController의 역할을 doDispatch()가 수행한다.

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {

    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    ModelAndView mv = null;

    // 1. 핸들러 조회
    mappedHandler = getHandler(processedRequest); 
    if (mappedHandler == null) {
        noHandlerFound(processedRequest, response);
        return;
    }

    // 2.핸들러 어댑터 조회-핸들러를 처리할 수 있는 어댑터
    HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

    // 3. 핸들러 어댑터 실행 -> 4. 핸들러 어댑터를 통해 핸들러 실행 -> 5. ModelAndView 반환 
    mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

    processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);    // render 호출
}

private void processDispatchResult(HttpServletRequest request, HttpServletResponse response, HandlerExecutionChain mappedHandler, ModelAndView mv, Exception exception) throws Exception {

    // 뷰 렌더링 호출
    render(mv, request, response);
}

protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {

    View view;  // View 인터페이스 선언
    String viewName = mv.getViewName(); // ModelAndView에서 논리 경로(viewName)을 뽑아낸다.

    //6. 뷰 리졸버를 통해서 뷰 찾기 ,7.View 반환
    view = resolveViewName(viewName, mv.getModelInternal(), locale, request);

    // 8. 뷰 렌더링
    view.render(mv.getModelInternal(), request, response);
}
```

## 3. 동작 순서
1. 핸들러 조회: 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러를 조회한다.
2. 핸들러 어댑터 조회: 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.
3. 핸들러 어댑터 실행: 핸들러 어댑터를 실행한다.
4. 핸들러 실행: 핸들러 어댑터가 실제 핸들러를 실행한다.
5. ModelAndView 반환: 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환
6. viewResolver 호출: 뷰 리졸버를 찾고 실행한다.
7. View 반환: 뷰 리졸버는 뷰의 논리 이름을 물리 이름으로 바꾸고, 렌더링 역할을 담당하는 뷰 객체를 반환한다.
8. 뷰 렌더링: 뷰를 통해서 뷰를 렌더링 한다.

## 4. 핸들러 매핑과 핸들러 어댑터
스프링 부트가 자동 등록하는 주요 핸들러 매핑과 핸들러 어댑터 (우선순위대로 수행, 낮을수록 높은 우선순위를 갖는다.)

### 4.1 HandlerMapping
- 0 = RequestMappingHandlerMapping: 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
- 1 = BeanNameUrlHandlerMapping: 스프링 빈의 이름으로 핸들러를 찾는다.

### 4.2 HandlerAdapter
- 0 = RequestMappingHandlerAdapter: 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
- 1 = HttpRequestHandlerAdapter: HttpReqestHandler 처리 (implements HttpRequestHandler)
- 2 = SimpleControllerHandlerAdapter: Controller 인터페이스 (과거에 사용, implements Controller)

### 4.3 BeanNameUrlHandlerMapping 과정
1. HandlerMapping을 순서대로 실행해서, BeanNameUrlHandlerMapping이 빈 이름으로 핸들러를 찾는다.
2. HandlerAdapter의 supports()에 핸들러를 인자로 넣고 해당 핸들러에 맞는 어댑터를 찾는다.
3. 디스패처 서블릿에서 조회한 HttpRequsetHandlerAdapter의 handle 메서드를 호출해 어댑터를 실행시키고 핸들러의 handleRequest 메서드를 호출해 핸들러를 실행시켜 ModelAndView를 반환한다.

## 5. 뷰 리졸버
- application.properties에 prefix 경로와 suffix 경로를 넣어주면 해당 url로 물리적 경로를 완성시킨다.
- 스프링 부트는 뷰 리졸버를 자동으로 빈으로 등록하는데 이때 prefix와 suffix 정보를 사용해서 등록한다.

### 5.1 스프링 부트가 자동 등록하는 주요 뷰 리졸버
- 1 = BeanNameViewResolver: 빈 이름으로 뷰를 찾아서 반환
- 2 = InternalResourceViewResolver: JSP를 처리할 수 있는 뷰를 반환

### 5.2 동작 과정
1. 핸들러 어댑터에서 핸들러를 실행시켜 논리 뷰 이름(경로)를 반환받는다.
2. 해당 논리 뷰 이름의 viewResolver를 순서대로 호출한다. (InternalResourceViewResolver의 경우)
3. InternalResourceResolver는 논리 경로를 물리 경로로 바꾸고 InternalResourceView를 반환
4. view.render()를 호출해 InternalResourceView는 forward()를 사용해서 JSP를 실행 (InternalResourceView는 view 인터페이스를 구현하고 있다.)

- InternalResourceViewResolver는 만약 JSTL 라이브러리가 있으면 InternalResourceView를 상속받은 JstView를 반환한다.
- JSP를 제외한 나머지 뷰 템플릿들은 forward()과정 없이 바로 렌더링 된다.
- Thymeleaf 뷰 템플릿을 사용하면 ThymeleafViewResolver를 등록해야한다.(최근에는 라이브러리를 추가하면 스프링 부트가 자동으로 등록해준다.)

## 6. @RequestMapping
- 핸들러 매핑: RequestMappingHandlerMapping
    + 스프링 빈 중에서 @RequestMapping또는 @Controller가 클래스 레벨에 붙어있는 경우에 매핑 정보르 인식한다.
- 핸들러 어댑터: RequestMappingHandlerAdapter

```java
@Controller
public class SpringMemberFormControllerV1 {

    @RequestMapping("/springmvc/v1/members/new-form")
    public ModelAndView process() {
        return new ModelAndView("new-form");
    }
}
```
- @Controller: 
    + 컴포넌트 스캔의 대상이 되어 스프링이 자동으로 스프링 빈으로 등록한다.
    + @Controller가 클래스 레벨에 붙어 있는 경우 매핑 정보로 인식한다.
- @RequestMapping: 요청 정보를 매핑한다.

### 6.1 동작 과정
1. 스프링 MVC는 애플리케이션 로딩시점에 @Controller가 붙어있는 모든 클래스를 찾아 내부의 @RequestMapping의 정보를 모두 보관한다.
2. 요청 url이 들어오면 @RequestMappingHandlerMapping이 getHandler(요청 url)로 해당 핸들러를 찾아 반환한다.
3. @RequestMappingHandlerAdapter가 해당 핸들러에 맞는 어댑터를 찾아 handle()을 호출해 핸들러의 @RequestMapping이 붙은 로직을 실행시켜 ModelAndView 객체를 반환한다.

### 6.2 mv.addObject("이름", 객체)
ModleAndView에 View에 출력할 데이터를 저장할 때 add.Object("이름", 객체)를 사용하면 코드를 더 줄일 수 있다.

## 7. 컨트롤러 통합
@RequestMapping은 클래스 단위 뿐만아니라 메서드 단위에도 사용 가능하기 때문에 핸들러들의 비즈니스 로직을 하나로 합치는 것도 가능하다.

### 7.1 매핑 url 중복 줄이기
클래스 레벨에 @RequestMapping("공통 경로")를 붙여주고 메서드 레벨에 @RequestMapping("상대 경로")를 붙여줌으로써 경로를 중복하지 않을 수 있다.

## 8. 실용적인 방식으로 리팩토링

### 8.1 ModelAndView 반환 대신 논리 경로만 반환
애노테이션 기반의 컨트롤러는 ModelAndView 객체로 반환해도 되고 논리 경로만 반환해도 되도록 유연하게 설계되어 있다.
- View에 출력할 데이터를 반환해야하는 경우 Model 객체를 이용해 데이터만 담는다.
```java
return "new-form"
```

### 8.2 애노테이션을 이용해 요청 데이터 직접 받기
- @RequestParam을 통해 HTTP 요청 파라미터를 직접 받을 수 있다.
- request.getParameter("username")와 거의 같은 코드이다.
- GET 쿼리 파리미터, POST Form 방식 모두 데이터를 파라미터 형식으로 전달하기 때문에 둘 다 적용 가능

```java
public String save(
    @RequestParam("username") String username,  // 요청 데이터 직접 받기
    @RequestParam("age") int age,
    Model model) {  // View에 출력할 데이터를 담을 객체

    Member member = new Member(username, age);
    memberRepository.save(member);

    model.addAttribute("member", member);       // model에 View에 출력할 데이터 담기
    return "save-result";
}
```

### 8.3 @GetMapping, @PostMapping
- 해당 로직이 HTTP 메서드도 구분 할 수 있도록 해준다. (GET 방식은 @GetMapping 애너테이션이 붙은 로직은 실행시킬 수 있지만 PostMapping 애너테이션이 붙은 로직은 실행 시킬 수 없다.)
- Get, Post, Put, Delete, Patch 모두 있다.
```java
// 옛날 방식
@RequestMapping(value = "/new-form", method = RequestMethod.GET)

// 현재
@GetMapping("/new-form")
```

## Reference
> 스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술 [김영한]

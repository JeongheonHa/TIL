# 로그인 처리 - 필터, 인터셉터
필터나 인터셉터로 로그인을 한 사용자만 해당 페이지로 들어갈 수 있게 할 수 있다.
- Http 헤더나 URL이 일치해야 실행되는 로직 같은 웹과 관련된 공통 관심사는 필터나 인터셉터를 사용하는 것이 좋다.
- 필터 - servlet 제공
- 인터셉터 - spring 제공
- 필터와 인터셉터 모두 싱글톤 객체이다.
## 1. 서블릿 필터

### 1.1 필터의 흐름
- 적절하지 않은 요청은 필터에서 걸러내 서블릿을 호출하지 않을 수 있다.
```html
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러 // 로그인 사용자
HTTP 요청 -> WAS -> 필터 // 비 로그인 사용자
```

### 1.2 필터 체인
필터는 체인으로 구성되어있어 중간에 자유롭게 추가할 수 있다.
- HTTP 요청 -> WAS -> `필터1` -> `필터2` -> `필터3` -> 서블릿 -> 컨트롤러

### 1.3 필터 인터페이스
Filter 인터페이스를 구현하면 서블릿 컨테이너가 필터를 싱글톤 객체로 생성하고 관리한다.
```java
public interface Filter {
    public default void init(FilterConfig filterConfig) throws ServletException {}  // default method
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException;
    public default void destroy() {} // default method
}
```
- `init()`: 필터 초기화 메서드, 서블릿 컨테이너가 생성될 때 호출된다.
- `doFilter()`: 요청이 올 때 마다 해당 메서드가 호출된다. 필터의 로직을 구현하면 된다. 
- `destroy()`: 필터 종료 메서드, 서블릿 컨테이너가 종료될 때 호출된다.

### 1.4 서블릿 필터 구현 - 모든 요청 로그
모든 요청 로그를 출력하는 필터 구현

#### 필터를 사용하려면 필터 인터페이스를 구현해야한다.
```java
@Slf4j
public class LogFilter implements Filter {
    // ...
}
```

#### 서블릿 컨테이너 생성시 로그 남기기
```java
@Override
public void init(FilterConfig filterConfig) throws ServletException {
    log.info("Log filter init");
}
```
#### 필터 로직을 수행하고 다음 필터 호출
- chain.dofilter(request, response): 다음 필터를 호출하고 필터가 없으면 서블릿(DispatcherServlet)을 호출한다.
```java
@Override
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
    log.info("log filter doFilter");

    HttpServletRequest httpRequest = (HttpServletRequest) request; // Http 요청 메세지의 정보를 얻고 싶으면 HttpServletRequest로 형변환 해야한다.
    String requestURI = httpRequest.getRequestURI();
    String uuid = UUID.randomUUID().toString();

    try {
        log.info("REQUEST [{}][{}]", uuid, requestURI);
        chain.doFilter(request, response);
    } catch (Exception e) {
        throw e;
    } finally {
        log.info("RESPONSE [{}][{}]", uuid, requestURI);
    }
}
```
#### 서블릿 컨테이너 종료시 로그 남기기
```java
@Override
public void destroy() {
    log.info("log filter destroy");
}

```

### 1.5 필터 등록
필터를 등록해야 사용할 수 있다.
- 스프링 부트를 사용하면 `FilterRegistrationBean`을 사용해서 등록하면 된다.
- `@ServletComponetScan`, `@WebFilter(filterName = "logFilter", urlPatterns = "/*")`을 이용해 등록할 수 있지만 순서 보장이 안되기 때문에 사용하지 말자.
- `addUrlPatterns("/*")`: 여러 패턴이 가능하지만 모든 url에 필터를 적용하고 whitelist로 몇개 만 접근할 수 있게 하는 것이 유지 보수에 좋다.

```java
@Configuration
public class WebConfig {
    @Bean // 수동으로 등록
    public FilterRegistrationBean logFilter() {
        FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
        filterRegistrationBean.setFilter(new LogFilter());  // 등록할 필터 지정
        filterRegistrationBean.setOrder(1); // 필터는 체인이기 때문에 동작 순서를 지정해 줘야한다. (낮을수록 먼저)
        filterRegistrationBean.addUrlPatterns("/*"); // 필터를 적용할 URL 패턴을 지정
        return filterRegistrationBean;
    } 
}
```

### 1.6 서블릿 필터 구현 - 인증 체크
로그인 되지 않은 사용자 해당 페이지에 접근하지 못하는 필터를 구현

#### whitelist
인증과 무관하게 항상 접근을 허용해야하는 경로들
```java
private static final String[] whitelist = {"/", "/members/add", "/login", "/logout", "/css/*"};
```

#### isLoginCheckPath
해당 경로가 whitelist에 없어 확인이 필요하면 true, 확인이 필요없으면 false
```java
private  boolean isLoginCheckPath(String requestURI) {
    return !PatternMatchUtils.simpleMatch(whitelist, requestURI);
}
```

#### doFilter
인증 체크 로직 수행
- `httpResponse.sendRedirect("/login?redirectURL=" + requestURI)` : 필터에 걸려 로그인 페이지로 리다이렉트될 때 요청 파라미터로 이전에 있었던 url을 담아서 리다이렉트 된다.
```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
    HttpServletRequest httpRequest = (HttpServletRequest) request;
    HttpServletResponse httpResponse = (HttpServletResponse) response;
    String requestURI = httpRequest.getRequestURI();

    try {
        log.info("인증 체크 필터 시작{}", requestURI); 
        if (isLoginCheckPath(requestURI)) { // 로그인을 확인해야하는 경로인지 확인
            log.info("인증 체크 로직 실행 {}", requestURI);
            HttpSession session = httpRequest.getSession(false); // 요청 메세지로부터 세션 가져오기

            if (session == null || session.getAttribute(SessionConst.LOGIN_MEMBER) == null) { // 세션이 없거나 또는 세션의 로그인 객체가 없는 경우
                log.info("미인증 사용자 요청 {}", requestURI);

                //로그인 페이지로 redirect
                httpResponse.sendRedirect("/login?redirectURL=" + requestURI);
                return; // 미 인증 사용자는 다음 필터로 넘어갈 수 없다.
            }

        }
        chain.doFilter(request, response);
    } catch (Exception e){
        throw e; // 예외가 터지고 여기서 처리하게 되면 정상 흐름처럼 동작하기 때문에 WAS까지 예외를 던져야한다.
    } finally {
        log.info("인증 체크 필터 종료 {}", requestURI);
    }
}
```

### 1.7 인증체크 필터 등록
WebConfig에 같은 방식으로 빈으로 수동으로 등록

### 1.8 로그인 성공시 이전 페이지로 redirect 구현

#### LoginController

```java
@PostMapping("/login")
public String loginV4(@Validated @ModelAttribute LoginForm form,
                        BindingResult bindingResult,
                        @RequestParam(defaultValue = "/") String redirectURL,
                        HttpServletRequest request) {
    if (bindingResult.hasErrors()) {
        return "login/loginForm";
    }

    Member loginMember = loginService.login(form.getLoginId(), form.getPassword());

    if (loginMember == null) {
        bindingResult.reject("loginFail", "아이디 또는 패스워드가 맞지 않습니다");
        return "login/loginForm";
    }

    // 로그인 성공
    HttpSession session = request.getSession();
    session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);

    return "redirect:" + redirectURL; // 이전 경로로 리다이렉트
}
```

### 1.9 request, response 바꾸기
chain.dofilter(request, response)로 다음 필터나 서블릿을 호출할 때 ServletRequest나 ServletResponse를 구현한 다른 객체를 넣어도 정상 수행된다. (잘 사용하지는 않는다.)

## 2. 스프링 인터셉터
서블릿 필터와 같이 웹과 관련된 공통 관심 사항을 효과적으로 해결할 수 있는 스프링 MVC가 제공하는 기술이다.
- 필터보다 더 좋다.

### 2.1 인터셉터 흐름
```html
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러 // 로그인 사용자
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 인터셉터  // 비 로그인 사용자
```

### 2.2 인터셉터 체인
필터는 체인으로 구성되어있어 중간에 자유롭게 추가할 수 있다.
```html
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 인터셉터1 -> 인터셉터2 -> 컨트롤러
```

### 2.3 인터셉터 인터페이스
```java
public interface HandlerInterceptor {
    default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {}
    default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {}
    default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {}
}
```
- 3가지 메서드 모두 Object 타입의 handler를 받기 때문에 모든 handler를 다 받을 수 있다.
- 아래의 그림을 보면 핸들러 어댑터에서 Object 타입의 handler를 인자로 받기 때문에 어떤 핸들러 객체든지 해당 핸들러 어댑터를 찾을 수 있다. 따라서 인터셉터에서 어떤 handler를 파라미터로 받아도 상관없다.

<img src = "https://user-images.githubusercontent.com/108064146/213511338-45c982f2-2e3b-4942-9b8c-7a4081d63d81.png">

#### 정상 흐름
<img src = "https://user-images.githubusercontent.com/108064146/213490137-96daadab-95aa-4e48-9e50-c14a4faf7d8f.png">

- `preHandle`: 핸들러 어댑터 호출 전에 호출
- `postHandle`: 핸들러 어댑터 호출 후에 호출, ModelAndView를 받을 수 있다.
- `afterCompletion`: 뷰가 렌더링된 이 후에 호출 (요청 완료 후), 예외를 파라미터로 받을 수 있다.

#### 예외 발생시 흐름
<img src = "https://user-images.githubusercontent.com/108064146/213489910-af905974-4b35-4226-bafc-ee1646cbdde3.JPG">

- `preHandle`: 핸들러 어댑터 호출 전에 호출
- `postHandle`: 컨트롤러에서 예외 발생시 호출 X
- `afterCompletion`: 뷰가 렌더링된 이 후에 호출 (요청 완료 후), 예외를 파라미터로 받을 수 있다.

#### afterCompletion
예외와 무관하게 공통 처리를 하려면 afterCompletion을 사용하면 된다.

### 2.4 인터셉터로 요청 로그 구현

#### 인터셉터를 만들려면 HandlerInterceptor를 구현한다.
```java
@Slf4j
public class LogInterceptor implements HandlerInterceptor {
    // 상수 등록
    public static final String LOG_ID = "logId";
    //...
}
```

#### preHandle
`request.setAttribute(LOG_ID, uuid)`: preHandle에서 지정한 값을 postHandle, afterCompletion에서도 사용하기 위해서 저장해 둔다.
```java
@Override
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    String requestURI = request.getRequestURI();
    String uuid = UUID.randomUUID().toString();

    request.setAttribute(LOG_ID, uuid); 

    log.info("REQUEST [{}][{}][{}]", uuid, requestURI, handler); // 인터셉터 시작 로그

    // 요청 로그 구현과 관련없는 로직
    if (handler instanceof HandlerMethod) {
        HandlerMethod hm = (HandlerMethod) handler; // 호출할 컨트롤러 메서드의 모든 정보가 저장된 목록
    }

    return true;
}
```

#### postHandle
```java
@Override
public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
    log.info("postHandle [{}]", modelAndView); // postHandle 호출 로그
}
```

#### afterCompletion
- afterCompletion은 예외가 발생해도 호출되기 때문에 postHandle이 아닌 여기에 종료 로그를 놓는다.
```java
@Override
public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
    String requestURI = request.getRequestURI();
    Object logId = request.getAttribute(LOG_ID);
    log.info("RESPONSE [{}][{}][{}]", logId, requestURI, handler); // 인터셉터 종료 로그
    if (ex != null) {
        log.error("afterCompletion error", ex);
    }
}
```

### 2.5 HandlerMethod vs RequestMappingHandlerMapping
- `HandlerMethod`: @Controller와 @RequestMapping을 활용한 핸들러 매핑은 @RequestMapping이 붙은 메서드를 HandlerMethod 객체 형태로 보관한다.
- `RequestMappingHandlerMapping`: 요청이 오면 HandlerMethod 객체로 변환하는 작업을 하거나 해당 요청에 매핑되는 HandlerMethod를 반환하는 작업을 한다. 따라서 스프링 컨테이너가 뜨면 RequestMappingHandlerMapping에 접근해서 저장된 매핑 정보와 핸들러 메서드 목록을 확인할 수 있다.
- `ResourceHttpRequestHandler`: @Controller가 아닌 `/resources/static`과 같은 `정적 리소스`가 호출되는 경우 ResourceHttpRequesthandler가 핸들러 정보로 넘어온다.

### 2.6 인터셉터 등록
- 여러 인터셉터를 등록할 경우 addInterceptors 메서드 안에 같은 형식으로 추가하면 된다.
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogInterceptor())
                .order(1)
                .addPathPatterns("/**") // 인터셉터에 걸릴 경로
                .excludePathPatterns("/css/**", "/*.ico", "/error"); // 인터셉터에 제외할 경로
    }
}
```

#### WebMvcConfigurer
WebMvcConfigurer를 구현하지 않으면 스프링부트의 기본 설정이 적용된다.
- WebMvcConfigurer 인터페이스를 구현하여 여러 default method들을 오버라이딩해서 사용하면 스프링부트의 기본 설정에 해당 메서드가 추가된다.
- WebMvcConfigurer를 구현하고 @Configuration 애노테이션을 붙이게 되면 스프링부트가 해당 설정파일을 읽고 구현된 내용을 설정에 추가한다. 이때 해당 메서드가 등록된다.

### 2.7 인터셉터 구현 - 인증 체크
서블릿 필터와 달리 예외 처리를 할 필요도 없고 whitelist를 지정할 필요도 없고 whitelist에 포함된 경로인지 확인할 필요도 없다. 단순히 preHandle 만 구현하면 된다.
```java
@Slf4j
public class LoginCheckInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        String requestURI = request.getRequestURI();

        log.info("인증 체크 인터셉트 실행 {}", requestURI);

        HttpSession session = request.getSession();

        if (session == null || session.getAttribute(SessionConst.LOGIN_MEMBER) == null) {
            log.info("미인증 사용자 요청");
            //로그인으로 redirect
            response.sendRedirect("/login?redirectURL=" + requestURI);
            return false;
        }
        return true;
    }
}
```

## 3. ArgumentResolver 활용
ArgumentResolver를 직접 만들어 기존의 로그인된 멤버를 session에서 찾아서 파라미터로 제공해주는 `@SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false)`애노테이션을 `@Login`애노테이션으로 바꿀 수 있다.

### 3.1 @Login 애노테이션 만들기
```java
@Target(ElementType.PARAMETER) // 파라미터에만 사용
@Retention(RetentionPolicy.RUNTIME) // 런타임까지 애노테이션 정보 유효
public @interface Login {
}
```

### 3.2 ArgumentResolver 만들기
HandlerMethodArguementResolver를 구현해서 만든다. <br>
추가 설명 : [HandlerMethodArgumentResolver](https://github.com/JeongheonHa/TIL/blob/main/Spring/spring-mvc/ch6-response.md#24-handlermethodargumentresolver)
```java
@Slf4j
public class LoginMemberArgumentResolver implements HandlerMethodArgumentResolver {

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        log.info("supportsParameter 실행");

        // @Login 애노테이션이 파리미터에 있는지 확인 
        boolean hasLoginAnnotation = parameter.hasParameterAnnotation(Login.class);
        // 파라미터 타입이 Member타입인지 확인
        boolean hasMemberType = Member.class.isAssignableFrom(parameter.getParameterType());
        // 둘다 만족하면 true
        return hasLoginAnnotation & hasMemberType;
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
                                NativeWebRequest webRequest, WebDataBinderFactory binderFactory) 
                                throws Exception {
        log.info("resolveArgument 실행");
        // session을 얻기 위해 HttpServletRequest를 뽑아온다.
        HttpServletRequest request = (HttpServletRequest) webRequest.getNativeRequest();
        HttpSession session = request.getSession(false);

        if (session == null) {
            return null;
        }
        // 세션에서 로그인된 객체를 반환
        return session.getAttribute(SessionConst.LOGIN_MEMBER);
    }
}
```
- `supportsParameter()`: 만족하면 해당 ArgumemtResolver를 사용
- `resolveArgument()`: 컨트롤러 호출 직전에 호출되어서 필요한 파라미터 정보를 반환

### 3.3 ArgumentResolver 등록
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        resolvers.add(new LoginMemberArgumentResolver());
    }
}
```

## Reference
> 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술 [김영한]

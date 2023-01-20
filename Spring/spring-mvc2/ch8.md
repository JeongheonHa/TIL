# 예외 처리

## 1. 서블릿의 예외 처리
1. Exception
2. response.sendError(HTTP 상태 코드, 오류 메세지)

### 1.1 Exception
요청이 오면 사용자 요청별로 별도의 쓰레드가 할당되고 서블릿 컨테이너 안에서 실행한다. <br>
이 때 예외 처리를 하지 않고 예외를 계속 던질 경우 WAS까지 전달된다.
```html
WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)
```
#### 동작 과정
1. 예외가 발생하고 예외를 던진다.
2. WAS까지 예외가 전달된다.
3. WAS는 Exception의 경우 서버 내부에서 처리할 수 없는 오류가 발생한 것으로 인식해서 HTTP 상태코드 500을 반환한다.

### 1.2 response.sendError(HTTP 상태 코드[, 오류 메세지])
호출한다고 해서 댕장 예외가 발생하는 것은 아니지만, 서블릿 컨테이너에게 오류가 발생했다는 점을 전달한다.

```html
WAS(sendError 호출 기록 확인) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러 (response.sendError())
```

#### 동작 과정
1. response.sendError()를 호출하면 response내부에 오류가 발생했다는 상태를 저장
2. 서블릿 컨테이너는 고객에게 응답 전에 response에 sendError()가 호출되었는지 확인
3. 호출되었다면 설정한 오류 코드에 맞춰 기본 오류 페이지를 보여준다.

### 1.3 서블릿이 제공하는 오류 화면 기능
서블릿은 Exception, response.sendError() 각각의 상황에 맞춘 오류 처리 기능을 제공한다.

#### 서블릿 오류 페이지 등록
예외 발생시 호출될 ErrorPage 객체를 factory에 등록
- `ErrorPage`: 예외나 상태코드에 맞는 sendError발생 시 해당 경로로 이동
```java
@Component
public class WebServerCustomizer implements WebServerFactoryCustomizer<ConfigurableWebServerFactory> {
    @Override
    public void customize(ConfigurableWebServerFactory factory) {
        ErrorPage errorPage404 = new ErrorPage(HttpStatus.NOT_FOUND, "/error-page/404"); // 상태코드가 NOT FOUND(400)이면 해당 경로로 이동
        ErrorPage errorPage500 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/error-page/500"); // 상태코드가 INTERNAL SERVER ERROR(500)이면 해당 경로로 이동
        ErrorPage errorPageEx = new ErrorPage(RuntimeException.class, "/error-page/500"); // 해당 예외 + 그 자손 예외가 발생하면 해당 경로로 이동
        factory.addErrorPages(errorPage404, errorPage500, errorPageEx);
    }
}
```

#### ErrorPageController 
오류 페이지 컨트롤러 구현
```java
@Controller
public class ErrorPageController {
    @RequestMapping("/error-page/404")
    public String errorPage404(HttpServletRequest request, HttpServletResponse response) {
        return "error-page/404";
    }
    @RequestMapping("/error-page/500")
    public String errorPage500(HttpServletRequest request, HttpServletResponse response) {
        return "error-page/500";
    }
}
```

### 1.4 오류 페이지 동작 원리

#### 예외 발생시 오류 페이지 요청 흐름
```html
1. WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)
2. WAS `/error-page/500` 다시 요청 -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러(/error- page/500) -> View
```
- 서버 내부에서 다시 WAS가 다시 요청하는 것으로 클라이언트는 무슨일이 발생하는지 알지 못한다.
- WAS는 오류 정보를 request의 attribute에 추가해서 넘겨준다.

```java
//request의 setAttribute로 추가하는 오류 정보
public static final String ERROR_EXCEPTION_TYPE = "javax.servlet.error.exception_type"; // 예외
public static final String ERROR_EXCEPTION = "javax.servlet.error.exception"; // 예외 타입
public static final String ERROR_MESSAGE = "javax.servlet.error.message"; // 오류 메세지
public static final String ERROR_REQUEST_URI = "javax.servlet.error.request_uri"; // 클라이언트 요청 URI
public static final String ERROR_SERVLET_NAME = "javax.servlet.error.servlet_name"; // 오류가 발생한 서블릿 이름
public static final String ERROR_STATUS_CODE = "javax.servlet.error.status_code"; // HTTP 상태 코드
```

### 1.5 오류 발생시 중복 호출 제거 (필터)
예외 발생시 필터나 인터셉터가 한번 더 호출되는 것은 매우 비효율적이다. 따라서 해당 요청이 클라이언트로 부터 받은 정상 요청인지 아니면 오류 페이지를 출력하기 위한 내부 요청인지 구분할 수 있어야한다. 필터의 경우 `DispatcherType`을 이용해서 구분할 수 있다.

### 1.6 DispatcherType

#### DispatcherType enum 클래스
```java
public enum DispatcherType {
    FORWARD, // 클라이언트 요청
    INCLUDE, // 오류 요청
    REQUEST, // 서블릿에서 다른 서블릿이나 JSP를 호출할 때
    ASYNC,   // 서블릿 비동기 호출
    ERROR    // 서블릿에서 다른 서블릿이나 JSP의 결과를 포함할 때
}
```

#### Filter에서 요청타입 로그 남기기
request.getDispatcherType(): 해당 요청 타입을 반환
```java
@Slf4j
public class LogFilter implements Filter {

   @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException { 
        try {
            log.info("REQUEST  [{}][{}][{}]", uuid, request.getDispatcherType(), requestURI);
            chain.doFilter(request, response);
        } catch (Exception e) {
            throw e;
        } finally {
            log.info("RESPONSE [{}][{}][{}]", uuid, request.getDispatcherType(), requestURI);
        }
    }
}
```

#### DispatcherType 등록
`setDispatcherTypes()`: 해당 타입이 있을 때만 필터가 호출된다.
- 지정해주지 않으면 기본값이 Disptacher.REQUEST로 클라이언트 요청시에만 호출된다.
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Bean
    public FilterRegistrationBean logFilter() {
    FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
    filterRegistrationBean.setDispatcherTypes(DispatcherType.REQUEST, DispatcherType.ERROR);
    return filterRegistrationBean;
    }
}
```

### 1.7 오류 발생시 중복 호출 제거 (인터셉터)

#### 인터셉터에서 요청타입 로그 남기기
```java
@Slf4j
public class LogInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        log.info("REQUEST  [{}][{}][{}][{}]", uuid, request.getDispatcherType(), requestURI, handler);
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
         log.info("RESPONSE [{}][{}][{}]", logId, request.getDispatcherType(), requestURI);

        // 에러 발생시 에러 로그 출력
        if (ex != null) {
            log.error("afterCompletion error!!", ex);
        }
    }
}
```

#### 인터셉터에서 중복되는 요청 제거
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogInterceptor())
        .excludePathPatterns(
                "/css/**", "/*.ico"
                , "/error", "/error-page/**" //오류 페이지 경로
        );
    }
}
```

## 2. 스프링 부트가 제공하는 오류 페이지 기능
1. `ErrorPage`를 자동으로 등록 (/error라는 경로를 기본 오류 페이지로 설정, 따라서 서블릿 밖에서 예외가 발생하거나 response.sendError(..)가 호출되면 모든 오류는 /error를 호출한다.)
2. `BasicErrorController`라는 스프링 컨트롤러를 자동으로 등록 (ErrorPage에서 등록한 /error를 매핑해서 처리하는 컨트롤러)
- 개발자는 오류 페이지 화면만 BasicErrorController가 제공하는 룰과 우선순위에 맞춰 등록하면된다.

### 2.1 BasicErrorController

#### 뷰 선택 우선순위
1. 뷰 템플릿
- resources/templates/error/500.html (구체적인게 우선)
- resources/templates/error/5xx.html (500번대는 해당 오류 페이지로 이동)
2. 정적 리소스
- resources/static/error/404.html
- resources/static/error/4xx.html
3. 적용 대상이 없을 때
- resources/templates/error.html

#### BasicErrorController가 제공하는 기본 정보들
BasicErrorController 컨트롤러는 아래의 정보를 model에 담아 뷰에 전달한다.
- 하지만, 고객에게 해당 오류들을 그대로 보여주는 것은 보안 문제가 될 수 있다.
- 따라서 고객에게는 간단한 오류 메세지를 보여주고 오류는 서버에서 로그로 남겨서 로그로 확인해야한다.
```html
- timestamp: Fri Feb 05 00:00:00 KST 2021
- status: 400
- error: Bad Request
- exception: org.springframework.validation.BindException 
- trace: 예외 trace
- message: Validation failed for object='data'. Error count: 1 
- errors: Errors(BindingResult)
- path: 클라이언트 요청 경로 (`/hello`)
```
##### application.properties
- 기본 값이 never인 부분은 never, always, on_param 옵션을 선택할 수 있다. (기본값을 사용하는 것을 권장)
- `never`: 사용 x
- `always`: 항상 사용
- `on_param`: 파라미터가 있을 때 사용
```html
server.error.include-exception=false
server.error.include-message=never
server.error.include-stacktrace=never
server.error.include-binding-errors=never
```

## Reference
> 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술 [김영한]

# API 예외 처리
API는 일반적인 예외 처리와는 달리 각 오류 상황에 맞는 오류 응답 스펙을 정하고 JSON으로 데이터를 내려주어야한다.

## 1. 서블릿 오류 페이지 방식의 API 예외 처리

### 1.1 WebServerCustomizer 동작

### 1.2 API 예외 컨트롤러
```java
@Slf4j
@RestController
public class ApiExceptionController {

    @GetMapping("/api/members/{id}")
    public MemberDto getMember(@PathVariable String id) {

        if (id.equals("ex")) {
            throw new RuntimeException("잘못된 사용자");
        }

        return new MemberDto(id, "hello " + id);
    }

    @Data
    @AllArgsConstructor
    static class MemberDto {
        private String memberId;
        private String name;
    }
}
```

### 1.3 ErrorPageController
오류 페이지 컨트롤러에 API 응답을 위해 HttpEntity로 반환하는 컨트롤러 메서드 생성
- 같은 url이지만 더 구체적인 컨트롤러 메서드가 먼저 실행된다.
```java
@RequestMapping(value = "/error-page/500", produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<Map<String, Object>> errorPage500Api(
            HttpServletRequest request, HttpServletResponse response) {

        Map<String, Object> result = new HashMap<>();
        Exception ex = (Exception) request.getAttribute(ERROR_EXCEPTION);
        result.put("status", request.getAttribute(ERROR_STATUS_CODE));
        result.put("message", ex.getMessage());

        Integer statusCode = (Integer) request.getAttribute(RequestDispatcher.ERROR_STATUS_CODE);
        return new ResponseEntity<>(result, HttpStatus.valueOf(statusCode));
    }
```

## 2. 스프링 부트가 제공하는 기본 오류 처리 방식의 API 예외 처리

### 2.1 BasicErrorController 살펴보기
- WebServerCustomizer는 주석
- `errorHtml()`: text/html인 경우 view 제공
- `error()`: 그 외의 경우 responseEntity로 JSON 데이터로 반환해서 제공
```java
@RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponseresponse) {}

@RequestMapping
public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {}
```

### 2.2 위의 API 예외처리의 문제점
BasicErrorController는 HTML 오류 페이지를 제공하는 경우에는 편리하지만 API 오류처리의 경우 각각의 컨트롤러나 예외마다 서로 다른 응답 결과를 출력해야 할 수 있기 때문에 서블릿과 스프링부트가 제공하는 방식은 좋은 방식이 아니다.

## 3. HandlerExceptionResolver를 이용한 API 예외 처리
오류 메세지, 형식등을 API마다 다르게 처리
- 기존에는 사용자가 입력을 잘못해서 오류가 발생한 경우에도 상태코드가 500으로 처리되었지만 상태코드가 400으로 처리되도록 변환

### 3.1 ApiExceptionController
```java
@GetMapping("/api/members/{id}")
public MemberDto getMember(@PathVariable("id") String id) {

    if (id.equals("bad")) {
        throw new IllegalArgumentException("잘못된 입력 값");
    }

    return new MemberDto(id, "hello " + id);
  }
```

### 3.2 HandlerExceptionResolver
스프링 MVC는 컨트롤러 밖으로 예외가 던져진 경우 예외를 해결하고, 동작을 새로 정의할 수 있도록 HandlerExceptionResolver를 제공한다.

<img src = "https://user-images.githubusercontent.com/108064146/214485632-3ec324a0-5b63-4feb-be6d-ce3aadc86109.JPG">
- postHandle()은 호출되지 않는다.

#### 인터페이스
```java
public interface HandlerExceptionResolver {
    ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex);
}
```

#### HandlerExceptionResolver 직접 만들기
- 예외 상태 코드 변환: 예외를 response.sendError()호출로 변경해서 서블릿에서 상태 코드에 따른 오류를 처리하도록 위임
- 뷰 템플릿 처리: ModelAndView에 값을 채워서 예외에 따른 새로운 오류 페이지를 렌더링해서 제공 (빈 값을 반환시 뷰 렌더링 x)
- API 응답 처리: 여기서 예외를 처리하고 HTTP응답 바디에 직접 데이터를 넣어주는 것도 가능
```java
@Slf4j
public class MyHandlerExceptionResolver implements HandlerExceptionResolver {
    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {

        try {
            if (ex instanceof IllegalArgumentException) {
                log.info("IllegalArgumentException resolver to 400");
                response.sendError(HttpServletResponse.SC_BAD_REQUEST, ex.getMessage()); // HttpServletResponse에 오류가 발생했다는 상태(상태코드, 오류 메세지)를 저장해둠
                return new ModelAndView(); // 빈 값으로 넘겨도 WAS까지 정상 흐름으로 리턴 된다. (경로가 없기 때문에 뷰 렌더링을 거치지 않는다.)
            }

        } catch (IOException e) {
            log.error("resolver ex", e);
        }

        return null; // null을 반환하면 다음 ExceptionResolver를 찾아서 실행
    }
}
```

#### WebConfig
ExceptionResolver를 사용하기 위해 등록
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        resolvers.add(new MyHandlerExceptionResolver());
    }
}
```

### 3.3 ExceptionResolver에서 예외 처리
예외를 여기서 처리해서 WAS까지 예외를 전달하지 않는다.

#### ApiExceptionController
예외 발생시 컨트롤러 메서드 추가
```java
@RestController
public class ApiExceptionController {
    if (id.equals("user-ex")) {
            throw new UserException("사용자 오류");
    }
    return new MemberDto(id, "hello " + id);
}
```

#### UserHandlerExceptionResolver 만들기
예외를 직접 처리하는 ExceptionResolver를 생성
```java
@Slf4j
public class UserHandlerExceptionResolver implements HandlerExceptionResolver {

    private final ObjectMapper objectMapper = new ObjectMapper();

    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        try {

            if (ex instanceof UserException) { // 예외 타입이 UserException이면
                log.info("UserException resolver to 400");
                String acceptHeader = request.getHeader("accept"); // 요청 메세지에서 Accept 헤더 값을 반환
                response.setStatus(HttpServletResponse.SC_BAD_REQUEST); // HttpServletResponse에 상태코드를 400으로 저장

                if ("application/json".equals(acceptHeader)) { // Accept 헤더가 json이라면 여기서 json 처리
                    Map<String, Object> errorResult = new HashMap<>();
                    errorResult.put("ex", ex.getClass());
                    errorResult.put("message", ex.getMessage());

                    String result = objectMapper.writeValueAsString(errorResult);

                    response.setContentType("application/json");
                    response.setCharacterEncoding("utf-8");
                    response.getWriter().write(result);

                    return new ModelAndView();
                } else {
                    //text/html의 경우
                    return new ModelAndView("error/500");
                }
            }
        } catch (IOException e) {
            log.error("resolver ex", e);
        }

        return null;
    }
}
```

#### WebConfig에 UserHandlerExceptionResolver 등록

## 4. 스프링이 제공하는 ExceptionResolver
HandlerExceptionResolverComposite에 다음 순서로 등록 (WebConfig 역할)
1. ExceptionHandlerExceptionResolver: @ExceptionHandler 처리 (대부분 이걸 사용)
2. ResponseStatusExceptionResolver: HTTP 상태 코드를 지정
3. DefaultHandlerExceptionResolver: 스프링 내부 기본 예외를 처리

### 4.1 ResponseStatusExceptionResolver
HTTP 상태 코드를 지정
#### @ResponseStatus
@ResponseStatus가 붙은 예외가 컨트롤러 밖으로 넘어가면 ResponseStatusExceptionResolver가 오류 코드를 지정한 코드로 변경하고 오류 메세지를 담는다.
- 결국 response.sendError()를 호출하는 것이다.
- 메세지 기능: `reason = "error.bad"`로 할 경우 MessageSource에서 error.bad에 매칭되는 메세지를 담는다.
```java
@ResponseStatus(code = HttpStatus.BAD_REQUEST, reason = "잘못된 요청 오류") 
public class BadRequestException extends RuntimeException {
}
```

#### @ResponseStatus의 단점
- @ResponseStatus는 개발자가 직접 변경할 수 없는 예외는 적용할 수 없다.
- 애노테이션을 사용하기 때문에 조건에 따라 동적으로 변경하는 것도 어렵다.

#### ResponseStatusException
동적으로 상태코드와 발생시킬 예외를 담은 ResponseStatusException 예외를 만들어 던짐으로써 ResponseStatusExceptionResolver가 오류 코드를 지정한 코드로 변경하고 오류 메세지를 담는다.
- @ResponseStatus로 해결할 수 없으면 ResponseStatusException을 사용하면 된다.
- RuntimeException의 자손 예외이다.
```java
@GetMapping("/api/response-status-ex2")
public String responseStatusEx2() {
    throw new ResponseStatusException(HttpStatus.NOT_FOUND, "error.bad", new IllegalArgumentException());
}
```

### 4.2 DefaultHandlerExceptionResolver
스프링 내부에서 발생하는 스프링 예외를 해결한다.
- 대표적으로 타입 오류 발생시 상태코드를 500에서 400으로 변경해준다.

```java
@GetMapping("/api/default-handler-ex")
public String defaultException(@RequestParam Integer data) {
    return "ok";
}
```

### 4.3 ExceptionHandlerExceptionResolver
@ExceptionHandler 처리
#### ErrorResult
예외가 발생했을 때 API 응답으로 사용하는 객체
```java
@Data
@AllArgsConstructor
public class ErrorResult {
    private String code;
    private String message;
}
```

#### ApiExceptionController
@ExceptionHandler 애노테이션을 선언하고, 해당 컨트롤러에서 처리하고 싶은 예외를 지정해준다. <br>
해당 컨트롤러에서 예외가 발생하면 해당 메서드가 호출된다.
- 지정한 예외와 그 자손 예외들 모두 해당되며, 해당 컨트롤러에 해당 예외와 자손 예외가 모두 존재하는 경우 자손 예외 발생시 더 구체적인 자손 예외가 우선순위를 갖는다.
```java
@ResponseStatus(HttpStatus.BAD_REQUEST)
@ExceptionHandler(IllegalArgumentException.class)
public ErrorResult illegalExHandle(IllegalArgumentException e) {
    log.error("[exceptionHandle] ex", e);
    return new ErrorResult("BAD", e.getMessage());
}
```

#### 실행 흐름
1. 컨트롤러 호출 결과 IllegalArgumentException 예외가 컨트롤러 밖으로 던져진다.
2. ExceptionHandlerExectionResolver가 실행
3. ExceptionHandlerExectionResolver가 해당 컨트롤러에 @ExcetionHandler가 있는지 확인
4. ilegalExHandle()를 실행
5. @ResponseBody가 적용되어 컨버터가 실행되고 JSON 형태로 응답 메세지에 반환
6. @ResponseStatus()로 지정한 상태코드로 응답한다.


#### 다양한 예외
다양한 예외를 한번에 처리 가능
```java
@ExceptionHandler({AException.class, BException.class})
public String ex(Exception e) {
    log.info("exception e", e);
}
```

##### 예외 생략
예외를 생략하면 메서드 파리미터의 예외가 지정된다.
```java
@ExceptionHandler
public ResponseEntity<ErrorResult> userExHandle(UserException e) {
    log.error("[exceptionHandle] ex", e);
    ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());
    return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);
}
```

#### 파라미터와 응답
@Exceptionhandler에는 다양한 파라미터와 응답을 지정할 수 있다.

#### HttpEntity 이용
@ResponseStatus와 달리 ResponseEntity를 사용하면 HTTP 응답 코드를 동적으로 변경할 수 있다.
```java
@ExceptionHandler
public ResponseEntity<ErrorResult> userExHandle(UserException e) {
    log.error("[exceptionHandle] ex", e);
    ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());
    return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);
}
```

## 5. @ControllerAdvice
@ControllerAdvice나 @RestControllerAdvice를 사용하면 정상 코드와 예외 코드를 분리할 수 있다.
- @ControllerAdvice는 대상(@Controller)으로 지정한 여러 컨트롤러에 @ExceptionHandler, @InitBinder 기능을 부여해주는 역할을 한다.
- 대상을 지정하지 않으면 모든 컨트롤러에 @ExceptionHandler, @InitBinder로 지정한 메서드들 적용.

### 5.1 @RestControllerAdvice 적용
```java
@Slf4j
@RestControllerAdvice
public class ExControllerAdvice {}
```

### 5.2 대상 컨트롤러 지정
```java
// 대상: RestController 애노테이션이 붙은 모든 컨트롤러들
@ControllerAdvice(annotations = RestController.class)
public class ExampleAdvice1 {}
// 대상: 해당 페키지 내에 있는 모든 컨트롤러들
@ControllerAdvice("org.example.controllers")
public class ExampleAdvice2 {}
// 대상: 특정 클래스에 할당할 수 있는 모든 컨트롤러들
@ControllerAdvice(assignableTypes = {ControllerInterface.class,
AbstractController.class})
public class ExampleAdvice3 {}
```

## Reference
> 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술 [김영한]

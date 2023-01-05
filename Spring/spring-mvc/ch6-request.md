# 스프링 MVC의 기본 기능

## 1. Jar vs War

### 1.1 War
톰캣 같은 WAS를 별도로 설치해서 빌드된 파일을 넣고 실행할 때 또는 JSP를 사용해야 할 때 사용한다. (물론 내장 서버도 사용 가능하다.)

### 1.2 Jar
항상 내장 서버(톰캣)를 사용할 때 사용한다.
- Jar를 사용하면 webapp을 사용할 수 없기 때문에 정적 리소스도 static 경로에 함께 포함한다.

- Jar를 사용하면 /resource/static/ 위치에 index.html파일을 두면 Welcome 페이지로 처리해준다.

## 2. Logging
- 스프링 부트 라이브러리에 로깅 라이브러리가 포함되어 있다.

### 2.1 SLF4J
로그 라이브러리를 통합해서 인터페이스로 제공하는 것이 SLF4J 라이브러리이다.

### 2.2 Logback
로그 라이브러리 중 하나로 SLF4J의 구현체이다.

### 2.3 로그 선언
```java
private Logger log = LoggerFactory.getLogger(getClass());

// 롬복 사용
@Slf4j
```

### 2.4 로그 호출
```java
log.trace("trace log={}", name);
log.debug("debug log={}", name);
log.info("info log={}", name);
log.warn(" warn log={}", name);
log.error("error log={}", name);
```

```java
// 시간                 로그 레벨 프로세스ID       쓰레드 명                    클래스 명                           로그 메세지
2023-01-04 22:28:54.406  INFO 11113 --- [nio-8080-exec-7] hello.springmvc.basic.LogTestController  : info log = Spring
```
- Level: TRACE > DEBUG > INFO > WARN > ERROR
- 개발 서버는 debug로 설정하고 운영 서버는 info로 설정하는 것이 좋다.
- 로그 레벨을 설정하기 위해선 application.properties에서 logging.level.hello.springmvc=debug 처럼 해당 패키지와 그 하위까지 로그 레벨을 설정할 수 있다.
- 기본 로그 레벨은 logging.level.root=info로 설정되어있다.
- 콘솔에서만 출력하는 것이 아닌 파일이나 네트워크 등 로그를 별도의 위치에 남길 수 있다.
- 파일로 남길 때는 일별, 특정 용량에 따라 로그를 분할하는 것도 가능
- 성능도 System.out보다 좋다.

## 3. 요청 매핑

### 3.1 @RestController
- @Controller는 반환 값이 String인 경우 뷰 이름으로 인식되기 때문에 @RestController는 반환 값이 String이어도 뷰를 찾는 것이 아닌 HTTP 메세지 바디에 바로 입력되도록 만든 것이다.
- 오류 모양도 Json 형태로 body에 넣는다.

### 3.2 @RequestMapping("경로")
- 해당 경로의 URL이 요청되면 해당 메서드가 실행되도록 매핑
- @RequestMapping({"/hello-basic", "/hello-go"}) 처럼 다중 설정이 가능해 둘 중 아무 URL로 와도 해당 메서드가 실행된다.

### 3.2 /{경로 변수}, PathVariable("경로 변수")
{userId}처럼 사용자가 경로 변수를 지정할 수 있도록 템플릿화 한 경우 파라미터에 @PathVariable(경로 변수)를 사용해 매칭되는 부분을 편리하게 조회할 수 있다.
- 경로 변수와 파라미터의 변수명이 같으면 @PathVariable String uesrId 로 파라미터내의 경로 변수를 생략할 수 있다.
```java
@GetMapping("/mapping/{userId}")
public String mappingPath(@PathVariable("userId") String data) {
    log.info("mappingPath userId={}", data);
    return "ok";
}
```

### 3.3 특정 파라미터 조건 매핑
요청 메세지의 특정 파라미터를 기반으로 매핑한다.
```java
@GetMapping(value = "/mapping-param", params = "mode=debug")
```
- "mode=debug" 같은 특정 파라미터 정보가 더 있어야 만 해당 메서드가 실행된다.
- 잘 사용 x

### 3.4 특정 헤더 조건 매핑
요청 메세지의 특정 헤더를 기반으로 매핑한다.
```java
@GetMapping(value = "/mapping-header", headers = "mode=debug")
```
- "mode=debug" 같은 특정 헤더 정보가 더 있어야 만 해당 메서드가 실행된다.

### 3.5 미디어 타입 조건 매핑
요청 메세지의 Content-Type 헤더를 기반으로 매핑한다. <br>

#### consumes
Content-Type에 따라 메서드를 분리하고 싶을 때 header 조건을 매핑하는 것이 아닌 타입 조건 매핑을 해야한다.
```java
@PostMapping(value = "/mapping-consume", consumes = "application/json")
```
- "application/json" 같은 특정 Content-Type에서 만 해당 메서드가 실행된다.

#### Accept, produce
Accept: 클라이언트가 특정 Content-Type만을 받을 수 있다고 요청 메세지의 헤더에 지정 <br>
produce: 응답 메세지 body를 특정 타입으로 만 생성

```java
@PostMapping(value = "/mapping-produce", produces = "text/html")
public String mappingProduces() {
    log.info("mappingProduces");
    return "ok";
}
```

## 4. HTTP 요청 - 헤더 조회
애노테이션 기반의 스프링 컨트롤러가 지원하는 다양한 파라미터
```java
public String headers (
HttpServletRequest request
HttpServletResponse response
HttpMethod httpMethod // HTTP 메서드 조회
Locale locale // 언어 조회 (우선순위가 놓은 거)
@RequestHeader MultiValueMap<String, String> headerMap // 모든 HTTP 헤더를 MultiValueMap 형식으로 조회 (하나의 키에 여러 값을 리스트 형태로 받는다, Map은 1:1 매핑)
@RequestHeader("host") String host // 특정 HTTP 헤더를 조회 (required, defaultValue)
@CookieValue(value = "myCookie", required = false) String cookie // 특정 쿠키 조회 (required, defaultValue)
) {}
```
> @Controller의 사용 가능한 파라미터 목록 <https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-arguments> <br>
> @Controller의 사용 가능한 응답 값 목록 <https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-return-types>

## 5. HTTP 요청 - 데이터 조회
- GET-쿼리 파라미터
- POST-HTML Form
- HTTP message body

### 5.1 요청 데이터 조회 - GET 쿼리 파라미터 & POST-HTML Form
두 요청 모두 데이터 형식이 `요청 파라미터`이기 때문에 HttpServletRequest의 request.getParameter()를 사용하면 모두 조회할 수 있다. <br>
이것을 요청 파라미터 조회하고 한다.

### 5.2 @RequestParam
- 파라미터 이름으로 요청 데이터를 바인딩 한다.
- request.getParameter("key")를 한 것과 같은 효과
```java
@ResponseBody
@RequestMapping("/request-param-v2")
public String requestParamV2(
        @RequestParam("username") String memberName,
        @RequestParam("age") int memberAge) {

    log.info("username={}, age={}", memberName, memberAge);
    return "ok";
}
```
#### @RequestParam의 괄호 생략 가능
HTTP 파라미터 이름이 변수 이름과 같으면 괄호를 생략할 수 있다.

#### @RequestParam 생략 가능
요청 데이터 타입이 String, int 등 단순 타입이면 @RequestParam을 생략할 수 있다.
- 단, 애노테이션을 생략하면 required=false가 적용된다.

#### @RequestParam(required = true/false)
- required = true: 해당 요청 파라미터를 무조건 보내야한다. (?username= 처럼 요청 값을 지정해주지 않으면 빈 문자로 인식하여 파라미터로 빈문자("")를 받는다.)
- required = false: 해당 요청 파라미터를 보내도 되고 안보내도 된다.
    + `@RequestParam(required = false) int age` 에서 null을 int타입에 저장하는 것은 불가능하므로 Integer로 변경하거나, defaultValue를 사용해야한다.

#### @RequestParam(defaultValue = " --- ")
파라미터에 값이 없는 경우 defaultValue로 지정한 값을 기본 값으로 지정할 수 있다.
- 요청 파라미터로 빈 문자가 와도 기본 값이 적용된다.

#### @RequestParam Map<String, Object> paramMap
요청 파라미터들을 Map에 key, value로 저장해서 사용할 수 있다.
- 요청 파라미터의 값이 1개의 key에 여러 value가 있는 경우 MultiValueMap을 사용하면 된다. (보통 1:1 매칭이기 때문에 MultiValueMap을 사용하는 일은 거의 없다.)

### 5.3 @ModelAttribute
@RequestParam은 요청 파라미터를 읽는 용도였다면 @ModelAttribute는 요청 파라미터를 받아 필요한 객체를 만들어준다.

```java
@ResponseBody
@RequestMapping("/model-attribute-v1")
public String modelAttributeV1(@ModelAttribute HelloData helloData) {
    log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
    return "ok";
}
```

#### 동작 과정
1. 지정된 타입의 객체를 생성 (해당 클래스 있어야함.)
2. 요청 파라미터의 이름으로 HelloData 객체의 프로퍼티(username, age ...)를 찾고 프로퍼티의 값을 변경하면 setXxxx가 호출되고, 조회하면 getXxxx가 호출된다.
3. 요청 데이터가 저장된 해당 객체를 파라미터로 제공해준다.

#### @ModelAttribute 생략 가능
스프링은 파라미터의 애노테이션을 생략하면 다음과 같은 규칙이 적용된다.
1. String, int, Integer 같은 단순 타입이면 @RequestParam 적용
2. argument resolver로 지정해둔 타입 외의 나머지 타입은 @ModelAttribute 적용

### 5.4 요청 데이터 조회 - HTTP message body
- HTTP API(rest API)에 주로 사용된다. (`JSON`, XML, TEXT)
- POST, PUT, PATCH
- @RequestParam, @ModelAttribute를 사용할 수 있다.

### 5.5 Input/Output Stream
- InputStream(Reader): HTTP 요청 메시지 바디의 내용을 직접 조회
- OutputStream(Writer): HTTP 응답 메시지의 바디에 직접 결과 출력
```java
@PostMapping("/request-body-string-v2")
public void requestBodyStringV2(InputStream inputStream, Writer responseWriter) throws IOException {
    String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
    log.info("messageBody={}", messageBody);
    responseWriter.write("ok");
}
```

### 5.6 HttpEntity (헤더 + 바디)
HTTP header, body 정보를 편리하게 조회할 수 있다.
- 요청: 메세지 헤더, 바디 정보를 직접 조회한다.
- 응답: 메세지 헤더, 바디 정보를 직접 반환한다. (반환 타입이 String이어도 view 이름 조회가 아닌 http 응답 메세지의 바디에 넣는다.)

```java
@PostMapping("/request-body-string-v3")
public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) {
    String messageBody = httpEntity.getBody();
    log.info("messageBody={}", messageBody);
    return new HttpEntity<>("ok");
}
```

#### 동작 과정
요청 메세지가 들어오면 HttpMessageConverter가 HTTP 메세지 바디의 내용을 우리가 원하는 문자나 객체 등으로 변환해준다. (incoding)
- 요청, 응답 모두에서 HttpMessageConverter가 사용된다.

#### RequestEntity
HttpEntity를 상속받아 추가적인 기능을 제공한다. (요청)
- HttpMethod, url 정보가 추가된다.

#### ResponseEntity
HttpEntity를 상속받아 추가적인 기능을 제공한다. (응답)
- HTTP 상태 코드를 설정할 수 있다.

### 5.7 @RequestBody (바디)
HTTP 메세지 바디 정보를 편리하게 조회할 수 있다.
- header 정보가 필요하다면 HttpEntity를 사용하거나 @RequestHeader를 사용하면된다.
- @RequestBody는 생략할 수 없다.

#### @ResponseBody
- @ResponeBody를 사용하면 응답 결과를 HTTP 메세지 바디에 직접 담아서 전달할 수 있다. (반환 타입이 String이어도 뷰 이름을 조회하지 않는다.)

### 5.8 JSON 형식으로 조회
HTTP 메세지 바디의 데이터를 JSON 형식으로 조회한다.

```java
@ResponseBody
@PostMapping("/request-body-json-v2")
public String requestBodyJsonV2(@RequestBody String messageBody) throws IOException {

    log.info("messageBody={}", messageBody);
    HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);
    log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());

    return "ok";
}
```
#### @RequestBody 특정 타입

```java
@ResponseBody
    @PostMapping("/request-body-json-v3")
    public String requestBodyJsonV3(@RequestBody HelloData helloData) throws IOException {
        log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
        return "ok";
    }
```

#### HttpMessageConverter
HttpEntity, @RequestBody, @ResponseBody를 사용하면 HttpMessageConverter가 Http 메세지 바디의 내용을 우리가 원하는 문자나 객체 등으로 변환해준다.
- 요청 메세지의 Content-Type이 application/json으로 들어오는 경우 JSON을 변환해주는 HttpMessageConverter가 실행된다. (objectMapper과정을 대신 해줌)
- 요청 메세지의 Accept에 application/json 타입만을 응답 받는다는 헤더를 설정하면 @ResponseBody는 응답 메세지 바디에 데이터를 넣을 때 JSON을 변환해주는 HttpMessageConverter가 실행된다. 따라서 데이터를 저장한 객체를 반환하면 JSON 형식으로 응답 메세지 바디에 직접 넣어줘서 JSON 형식으로 응답 받을 수 있다.

- @RequestBody 요청: JSON 요청 -> HttpMessageConverter -> 객체
- @ResponseBody 응답: 객체 -> HttpMessageConverter -> JSON 응답

### 5.9 최종 형태
```java
// 요청 파라미터 데이터 조회
@ResponseBody
@RequestMapping("/request-param-default")
public String requestParamDefault(
        @RequestParam(defaultValue = "guest") String username,
        @RequestParam(defaultValue = "-1") int age) {

    log.info("username={}, age={}", username, age);
    return "ok";
}

// 요청 파라미터 데이터 객체에 담아서 조회
@ResponseBody
@RequestMapping("/model-attribute-v1")
public String modelAttributeV1(@ModelAttribute HelloData helloData) {

    log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
    return "ok";
}

// HTTP 메세지 바디의 데이터 조회
@ResponseBody
@PostMapping("/request-body-string-v4")
public String requestBodyStringV4(@RequestBody String messageBody) throws IOException {

    log.info("messageBody={}", messageBody);
    return "ok";
}

// JSON 형식의 데이터 조회
// Content-Type: application/json
@ResponseBody
@PostMapping("/request-body-json-v3")
public String requestBodyJsonV3(@RequestBody HelloData helloData) throws IOException {
    log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
    return "ok";
}
```

## Reference
> 스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술 [김영한]

# HTTP 응답
- 정적 리소스: 정적인 리소스를 제공할 때 사용
- 뷰 템플릿 사용: 동적인 리소스를 제공할 때 사용
- HTTP 메세지 사용: HTTP API를 제공하는 경우 (HTML x, 데이터 o(json))

## 1. 정적 리소스
해당 파일을 변경없이 그대로 서비스하는 것
- 스프링 부트가 지원하는 정적 리소스 디렉토리: /static, /public, /resourecs, /META-INF/resources
- src/main/resources는 리소스를 보관하는 곳이고, 클래스 패스의 시작 경로이다. 해당 디렉토리에 리소스를 넣어두면 스프링 부트가 정적 리소스로 서비스를 제공한다.

## 2. 뷰 템플릿
뷰 템플릿을 거쳐 HTML을 생성하고, 뷰가 응답을 만들어서 전달한다.
- 뷰 템플릿이 만들 수 있는 모든 것이 가능하며 주로 HTML을 동적으로 생성하는데 사용한다.
- 뷰 템플릿 경로: src/main/resources/templates

### 2.1 뷰 템플릿을 호출하는 컨트롤러
```java
@RequestMapping("/response-view-v2")
public String responseViewV2(Model model) {
    model.addAttribute("data", "hello!");
    return "response/hello";
}
```
- @ResponseBody, HttpEntity를 사용하면, 뷰 템플릿을 사용하는 것이 아닌, HTTP 응답 메세지 바디에 직접 데이터를 넣을 수 있다. (문자열, 객체 모두)

## 3. HTTP API
HTML이 아닌 데이터를 직접 전달해야 하므로, 응답 메세지 바디에 데이터를 JSON 형식으로 실어 보낸다.
- Http 메세지 바디에 데이터가 있으면 무조건 Content-Type을 지정해 줘야한다. (요청, 응답 모두)

```java
// 텍스트 형식으로 응답 - HttpEntity 이용
@GetMapping("/response-body-string-v2")
public ResponseEntity<String> responseBodyV2() {
    return new ResponseEntity<>("ok", HttpStatus.OK);
}

// 텍스트 형식으로 응답 - @ResponseBody
@GetMapping("/response-body-string-v3")
public String responseBodyV3() {
    return "ok";
}

// 
// Json 형식으로 응답 - HttpEntity 이용
@GetMapping("/response-body-json-v1")
public ResponseEntity<HelloData> responseBodyJsonV1() {
    HelloData helloData = new HelloData();  // 요청 데이터를 해당 객체에 담아 파라미터로 전달 받을 수 있다. (컨버터 이용)
    helloData.setUsername("userA");
    helloData.setAge(20);
    return new ResponseEntity<>(helloData, HttpStatus.OK);
}

// Json 형식으로 응답 - @ResponseBody 이용
@ResponseStatus(HttpStatus.OK)  // 상태 코드 지정
@ResponseBody
@GetMapping("/response-body-json-v2")
public HelloData responseBodyJsonV2() {
    HelloData helloData = new HelloData();
    helloData.setUsername("userA");
    helloData.setAge(20);
    return helloData;
}
```
- 클래스 레벨에 @ResponseBody를 적용하면 모든 메서드에 적용할 수 있으며, @RestController를 사용하면 @ResponseBody와 @Controller를 동시에 적용할 수 있는 가 있다.

# HttpMessageConverter
Http 메세지 바디의 내용을 우리가 원하는 문자나 객체 등으로 변환해준다.

- 응답의 경우 클라이언트의 HTTP Accept header와 서버 컨트롤러의 반환 타입을 고려해 컨버터가 선택된다.
- HTTP 요청: @RequestBody, @HttpEntity(RequestEntity) 에서 HttpMessageConverter 가 실행된다.
- HTTP 응답: @ResponseBody, @HttpEntity(ResponseEntity) 에서 HttpMessageConverter 가 실행된다.

## 1. 메서드
- canRead(), canWrite(): 메세지 컨버터가 해당 클래스, 미디어 타입을 지원하는지 체크한다.
- read(), write(): 메세지 컨버터를 통해 메세지를 읽고 쓰는 기능을 한다.

## 2. 스프링 부트의 주요 메세지 컨버터
스프링 부트는 대상 클래스 타입과 미디어 타입 둘을 체크해서 사용여부를 결정한다. (우선순위 대로)
- 0 = ByteArrayHttpMessageConverter
- 1 = StringHttpMessageConverter
- 2 = MappingJackson2HttpMessageConverter

### 2.1 ByteArrayHttpMessageConverter
byte[] 데이터를 처리한다.
- 클래스 타입: byte[] , 미디어타입: */*
- 요청 예) @RequestBody byte[] data
- 응답 예) @ResponseBody return byte[], 쓰기 미디어타입: application/octet-stream

### 2.2 StringHttpMessageConverter
String 문자로 데이터를 처리한다. 
- 클래스 타입: String , 미디어타입: */*
- 요청 예) @RequestBody String data
- 응답 예) @ResponseBody return "ok", 쓰기 미디어타입: text/plain

### 2.3 MappingJackson2HttpMessageConverter
application/json 데이터를 처리한다.
클래스 타입: 객체 또는 HashMap , 미디어타입 application/json 관련
요청 예) @RequestBody HelloData data
응답 예) @ResponseBody return helloData, 쓰기 미디어타입: application/json 관련

> 미디어 타입: Content-Type로 */*는 모든 타입이 가능하다는 뜻이다.

### 2.4 HandlerMethodArgumentResolver
요청 값을 변환하고 처리한다.
- 애노테이션 기반 컨트롤러를 처리하는 RequestMappingHandlerAdapter는 ArgumentResolver를 호출해서 컨트롤러가 필요로 하는 다양한 파라미터의 객체를 생성하고 컨트롤러를 호출하면서 값을 넘겨준다.

### 2.5 HandlerMethodReturnHandler
ArgumentResolver와 비슷하지만 이것은 응답 값을 변환하고 처리한다.

<img src="https://user-images.githubusercontent.com/108064146/210949028-bdf32e53-f95c-45ab-9f53-bdd6f83a7a2c.png">


### 2.6 ArgumetResolver, ReturnHandler 상속 관계
서로 다른 요청, 응답 인터페이스여도 하나의 클래스로 사용하는 것을 확인 할 수 있다.
- `@RequestBody`, `@ResponseBody`가 있으면 RequestResponseBodyMethodProcessor를 사용
- `HttpEntity`가 있으면 HttpEntityMethodProcessor를 사용

<img src="https://user-images.githubusercontent.com/108064146/210950607-fb06f99c-8ad0-4d4e-a909-7a69ce50e170.png">

<img src="https://user-images.githubusercontent.com/108064146/210950913-9259b103-23fd-4fb6-8c11-a2095cd6d42c.png">


### 2.6 동작 과정

#### HTTP 요청 데이터를 읽는 경우
1. 컨트롤러에 @RequestBody, HttpEntity 파라미터를 사용하면
2. HTTP 요청이 오면 핸들러 어댑터가 컨트롤러의 파라미터 정보를 가져온다.
3. 핸들러 어댑터는 ArgumentResolver의 supportParameter()를 호출해서 해당 파라미터를 지원하는지 체크한다.(true/false)
4. 지원하면 resolveArgument()를 호출해서 HttpMessageConverter를 통해 요청 메세지 바디의 데이터를 읽어와 생성된 객체를 반환한다.
5. 핸들러 어댑터는 컨트롤러 호출시에 생성된 객체를 인자로 보내준다.

##### 컨버터가 요청 메세지 바디의 데이터를 읽어오는 동작 과정
1. 메세지 컨버터가 메세지를 읽을 수 있는지 확인하기 위해 canRead()를 호출한다.
    - 대상 클래스 파라미터 타입을 확인하여 컨버터를 선정
    - 해당 컨버터가 HTTP 요청으로 온 Content-Type(미디어 타입)을 지원하는지 확인
2. canRead()를 만족하면 read()를 호출해서 객체를 생성하고 resolveArgument()에 반환한다.

#### HTTP 응답 데이터를 생성하는 경우
1. 컨트롤러에서 @ResponseBody, HttpEntity로 핸들러 어댑터에 값을 반환하면
2. 핸들러 어댑터는 ReturnValueHandler의 supportReturnType()을 호출해서 해당 반환 타입을 지원하는지 체크한다.(true/false)
3. 지원하면 handleReturnValue()를 호출해서 HttpMessageConverter를 통해 HTTP 응답 메세지 바디에 데이터를 생성한다.

##### 컨버터가 응답 메세지 바디에 데이터를 생성하는 동작 과정
1. 메세지 컨버터가 메세지를 쓸 수 있는지 확인하기 위해 canWrite()를 호출한다.
    - 대상 클래스의 return 타입을 확인하여 컨버터를 선정
    - HTTP 요청의 Accept의 타입을 해당 컨버터가 지원하는지 확인
2. canWrite()조건을 만족하면 write()를 호출해서 HTTP 응답 메세지 바디에 데이터를 생성한다.

> HttpMessageConverter, HandlerMethodArgumentResolver, HandlerMethodReturnValueHandler 모두 인터페이스라는거 생각하자. 스프링에서 제공하는 기능들은 대부분 인터페이스로 추상화되어있다.

## Reference
> 스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술 [김영한]

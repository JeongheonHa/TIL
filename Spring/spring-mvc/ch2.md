# Servlet

## 1. 동작 방식
1. 스프링 부트가 실행되면서 내장 톰켓 서버를 띄워준다. 
2. 톰켓 서버 내의 서블릿 컨테이너에서 helloServlet을 생성한다.
3. WAS에서 웹 브라우저에서 요청한 http 메세지를 읽어서 requset와 response 객체를 만든다.
4. 서블릿 컨테이너에 싱글톤으로 떠있는 helloServlet을 호출하고 service 메서드를 호출해서 request, response 객체를 service 메서드에 넘겨준다.
5. 메세드가 종료되면서 AWS에서 response객체를 기반으로 http 응답 메세지를 생성해서 웹 브라우저에 응답
6. 웹 브라우저는 응답 메서지를 렌더링해서 화면에 출력


### 1.1 스프링 부트 서블릿 환경 구성 (1)
스프링 부트는 서블릿을 직접 등록해서 사용할 수 있도록 @ServletComponentScan을 지원한다. @WebServlet애너테이션을 찾아 해당 이름의 서블릿 객체를 생성하고 WAS에서 요청 메세지를 기반으로 request와 response 객체를 생성한다.
```java
@ServletComponentScan
@SpringBootApplication
public class ServletApplication {

	public static void main(String[] args) {
		SpringApplication.run(ServletApplication.class, args);
	}

}
```

### 1.2 @WebServlet (서블릿 애노테이션) (2~5)
HTTP 요청을 통해 매핑된 URL이 호출되면 서블릿 컨테이너는 service 메서드를 실행한다.
```java
@WebServlet(name = "helloServlet", urlPatterns = "/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    }
}
```

## 2. 추가 설정

### 2.1 HTTP 요청 메세지 로그 확인하기
application.properties에 해당 설정을 추가함으로써 서버가 받은 요청 메세지를 출력해준다. (단, 개발 성능을 저하시킬 수 있으므로 개발 단계에서만 적용하도록 하자.)
```java
logging.level.org.apache.coyote.http11=debug
```

### 2.2 welcome 페이지
내 도메인에 왔을 때 첫 화면을 welcome 페이지라고 한다.
- webapp경로에 index.html을 두면 http://localhost:8080 호출시 index.html이 열린다.

## 3. HttpServletRequset
HTTP 요청 메세지를 편리하게 사용할 수 있도록 대신해서 HTTP 요청 메세지를 파싱하여 HttpServletRequest 객체에 담아서 제공해준다.

### 3.1 HttpServletRequest의 추가 부가기능

#### 임시 저장소 기능
해당 HTTP 요청이 시작부터 끝날 때 까지 유지되는 임시 저장소 기능을 제공한다.
- 저장: request.setAttribute(name, value)
- 조회: request.getAttrebute(name)

#### 세션 관리 기능
- request.getSession(create: true)

#### 데이터를 편리하게 읽을 수 있는 기능
- request.getParameter("key")

#### message body를 통으로 읽어드리는 기능
- Json 같은 형식의 데이터를 통으로 읽어드릴 수 있다.

### 3.1 start-line 조회
- request.`getMethod()`      // GET
- request.`getProtocol()`    // HTTP/1.1
- request.`getScheme()`      // http
- request.`getRequestURL()`  // http: //localhost:8080/request-header
- request.`getRequestURI()`  // /request-header
- request.`getQueryString()` // username=hi
- request.`isSecure(`)`      // https 사용 유무

### 3.2 header 모든 정보 조회
```java
request.getHeaderNames().asIterator()
        .forEachRemaining(headerName -> System.out.println(headerName + ":" + request.getHeader(headerName)));
```

### 3.3 header 편리한 조회
- getServerName() // localhost
- getServerPort() // 8080

#### Accept-Language 편의 조회
```java
request.getLocales().asIterator()
        .forEachRemaining(locale -> System.out.println("locale = " + locale));

System.out.println("request.getLocale() = " + request.getLocale());

// 결과
// locale = ko
// locale = en_US
// locale = en
// locale = ko_KR
// request.getLocale() = ko
```

#### cookie 편의 조회
```java
if (request.getCookies() != null) {
    for (Cookie cookie : request.getCookies()) {
        System.out.println(cookie.getName() + ": " + cookie.getValue());
    }
}
```

#### Content 편의 조회
- request.getContentType()       // message body 데이터 타입 조회
- request.getContentLength()     // ContentLength 정보 조회
- request.getCharacterEncoding() // UTF-8, 인코딩 정보 조회

#### 기타 정보 조회

##### remote 정보
http 요청 메세지에 오는 것이 아닌 내부에서 네트워크 커넥션이 맺어진 정보를 조회하는 방법
- request.getRemoteHost()   // 0:0:0:0:0:0:0:1
- request.getRemoteAddr()   // 0:0:0:0:0:0:0:1
- request.getRemotePort()   // 54305

##### local 정보
내 서버에 대한 정보
- request.getLocalName() = localhost 
- request.getLocalAddr() = 0:0:0:0:0:0:0:1 
- request.getLocalPort() = 8080

## 4. HTTP 요청 데이터 전달 방법 3가지
- GET-쿼리 파리미터
- POST-HTML Form
- HTTP message body

### 4.1 GET-쿼리 파리미터
메세지 바디 없이 URL의 쿼리 파라미터를 이용해 데이터를 전달 하는 방식
- http://localhost:8080/request-param?username=hello&age=20
- key: username, age
- value: hello, 20

#### GET-쿼리 파리미터 방식으로 온 요청 메세지의 데이터를 조회하는 메서드
```java
// 전체 파라미터 조회
request.getParameterNames().asIterator()
        .forEachRemaining(paraName -> System.out.println(paraName + 
        "=" + request.getParameter(paraName)));   

//단일 파라미터 조회 
String username = request.getParameter("username"); 
//파라미터 이름들 모두 조회
Enumeration<String> parameterNames = request.getParameterNames(); 
//파라미터를 Map 으로 조회
Map<String, String[]> parameterMap = request.getParameterMap(); 
//이름이 같은 복수 파라미터 조회
String[] usernames = request.getParameterValues("username"); 
```
- 이름이 같은 복수 파라미터에서 request.getParameter("username")를 사용하면 request.getParameterValues()의 첫 번째 값을 반환한다.

### 4.2 POST HTML-Form
HTML의 Form 형식을 사용해서 서버로 데이터를 요청하는 방식 (주로 회원가입, 상품 주문 등에서 많이 사용)
- `content-type: application/x-www-form-urlencoded` 타입의 데이터 형식을 사용하며 해당 타입은 GET방식에서 쿼리 파라미터 형식과 같기 때문에 쿼리 파라미터 조회 메서드를 그대로 사용하면된다. ( getParameter("username")... )
- 메세지 바디에 쿼리 파라미터 형식으로 데이터를 전달한다.

### 4.3 HTTP message body
HTTP 요청 메세지의 메세지 바디에 데이터를 직접 담아서 요청하는 방식
- 주로 HTTP API(rest API)에서 주로 사용한다.
- 데이터의 형식은 JSON, XML, TEXT가 있지만 대부분 JSON 방식을 사용한다.
- 메세지 바디를 사용하기 때문에 POST, PUT, PATCH 메서드 방식을 사용할 수 있다.

#### http 요청 메세지 형태 (Json)

```java
POST http://localhost:8080/request-body-json 
content-type: application/json
message body: {"username": "hello", "age": 20} 
결과: messageBody = {"username": "hello", "age": 20}
```

#### 메세지 바디의 데이터 조회 (Json)
- Json 형식을 파싱할 수 있도록 객체를 하나 생성 (객체에 Json 형태로 온 데이터를 대입)
```java
@Getter @Setter
public class HelloData {

    private String username;
    private int age;
}
```

- Json 형식으로 데이터 조회 <br>
Json 결과를 파싱해서 사용할 수 있는 자바 객체로 변환하려면 Jackson, Gson 같은 Json 변환 라이브러리를 추가해서 사용해야하며 스프링 부트로 Spring MVC를 선택하면 기본으로 Jackson 라이브러리를 함께 제공한다.
```java
// Jackson 라이브러리
private ObjectMapper objectMapper = new ObjectMapper();

// http 메세지 바디의 내용을 바이트 코드로 조회
ServletInputStream inputStream = request.getInputStream();

// 바이트 코드의 데이터를 지정한 incoding 정보를 바탕으로 문자열로 조회
String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

// readValue: Json 형식의 message body 데이터를 파싱해서 HelloData 객체로 변환
HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);

System.out.println("helloData.username = " + helloData.getUsername());
System.out.println("helloData.age = " + helloData.getAge());
```

## 5. HttpServletResponse
HTTP 응답 메세지를 생성하고 편의 기능(Content-Type, 쿠키, Redirect)을 제공한다.

### 5.1 HTTP 응답 메세지 생성
#### status-line
- response.setStatus(HttpservletResponse.SC_OK);    // 상태 코드를 200으로 설정

#### header
- response.setHeader("Content-Type", "text/plain;charset=utf-8");   // 메세지 바디의 데이터 타입과 incoding 정보 설정
- response.setHeader("Cache-Control", "no-cache, no-store, must-revalidate");   // cache 완전 무효화
- response.setHeader("Pragma", "no-cache"); // cache 완전 무효화
- response.setHeader("my-header","hello");  // 새로운 header 생성

#### header 더 편하게 생성하기
- response.setContentType("text/plain");
- response.setCharacterEncoding("utf-8");
- response.setContentLength(2); // 생략시 자동 생성 (생략해서 사용 추천)

#### header - cookie
```java
Cookie cookie = new Cookie("myCookie", "good"); // 쿠키 생성
cookie.setMaxAge(600);  // 시간 제한 설정
response.addCookie(cookie); // 응답 메세지의 header에 쿠키 넣기
```

#### header - redirect
```java
response.setStatus(HttpServletResponse.SC_FOUND); //302
response.setHeader("Location", "/basic/hello-form.html"); // redirect할 Location을 /basic/hello-form.html로 지정

// 더 줄인 코드 (자동으로 302 상태코드 추가)
response.sendRedirect("/basic/hello-form.html");
```

#### message body
- 단순 텍스트 응답 (writer.prointln("ok"))
- HTML 응답
- HTTP API (rest API, Json 방식 응답)

#### message body - 단순 텍스트 응답
```java
PrintWriter writer = response.getWriter();
writer.println("ok");   // ok + enter (ContentLength = 3)
```

#### message body - HTML 응답
```java
response.setContentType("text/html");
response.setCharacterEncoding("utf-8");

PrintWriter writer = response.getWriter();
writer.println("<html>");
writer.println("<body>");
writer.println("<div>안녕하세요</div>");
writer.println("</body>");
writer.println("</html>");
```

#### message body - HTTP API
```java
response.setContentType("application/json");
response.setCharacterEncoding("utf-8");

// 데이터가될 객체 생성
HelloData helloData = new HelloData();
helloData.setUsername("kim");
helloData.setAge(20);

// helloData 객체를 Json 문자로 변경 
String result = objectMapper.writeValueAsString(helloData); // {"username":"kim", "age":20}
response.getWriter().write(result);
```

## Reference
> 스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술 [김영한]

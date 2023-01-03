# 서블릿, JSP, MVC 패턴으로 웹 만들기

## 1. 서블릿 만으로 만들기
서블릿 만으로 웹을 만들경우 비즈니스 로직을 만들고 HTTP 요청메세지를 처리하는 부분은 수월하지만 Form 형식으로 응답 메세지를 만드는 부분은 굉자히 힘들다.

## 2. JSP 만으로 만들기
Form을 작성하는 부분은 수월해졌지만 JSP에 비즈니스 로직까지 함께 작성함에 따라 JSP가 너무 많은 역할을 수행한다.

## 3. MVC 패턴
비즈니스 로직은 서블릿 처럼 다른 곳에서 처리하고, JSP는 목적에 맞게 HTML로 화면(View)을 그리는 일에 집중한다.
- 컨트롤러: HTTP 요청을 받아 파라미터를 검증하고, 비즈니스 로직을 실행하며 View에 전달할 결과 데이터를 조회해서 모델에 담는다.
    + 일반적으로 비즈니스 로직은 서비스라는 계층을 별도로 만들어 처리한다.
- 모델: View에 출력할 데이터를 담아둔다.
- 뷰: 모델에 담겨있는 데이터를 사용해서 화면을 그리는 일에 집중한다.

### 3.1 MVC 패턴 적용
- servlet을 controller로 사용하고 JSP를 View로 사용하고 Model은 HttpServletRequest객체의 내부 저장소를 사용한다.

#### 뷰 렌더링 과정 구현
```java
String viewPath = "/WEB-INF/views/new-form.jsp";
RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);  // 해당 경로로 이동할 수 있는 객체를 생성
dispatcher.forward(request, response);  // 다른 서블릿이나 JSP이동하는 기능을 가졌으며 서버 내부에서 다시 호출이 발생한다.
```
#### webapp
webapp 내의 파일은 controller를 거치지 않고 url에 경로를 넣어주면 직접 접근이 가능하다. <br>
webapp 디렉토리는 자동으로 호출되기 때문에 webapp아래에 index.html을 만들어 welcome page로 정할 수 있다.

#### webapp/WEB-INF
/WEB_INF 경로안에 JSP가 있으면 외부에서 직접 JSP를 호출할 수 없고 Controller를 거쳐야만 접근이 가능하다. <br>
url에 경로를 넣어줘도 응답하지 않는다.

#### MVC 패턴을 적용한 후 단점
- 공통 처리가 어렵다.
- 뷰 렌더링 과정이 중복된다.
- viewPath가 중복된다.
- 사용하지 않는 코드가 존재한다. (request, response ...)

# MVC 프레임워크 만들기
MVC 패턴을 적용한 후 단점으로 보인 것들을 리팩토링 해보자.

## 1. v1
FrontController 생성
### 1.1 FrontController
프론트 컨트롤러 서블릿 하나로 클라이언트의 요청을 받아 요청에 맞는 컨트롤러를 찾아서 호출해준다.
- 공통 처리 가능
- 프론트 컨트롤러를 제외한 나머지 컨트롤러는 서블릿을 사용하지 않아도 된다.
- 스프링 웹 MVC의 DispatcherServlet이 FrontController 패턴으로 구현되어있다.

### 1.2 과정
1. http 요청이 오면 FrontController에서 요청 메세지의 URI를 뽑아낸다.
2. 해당 URI와 컨트롤러들의 Mapping 정보를 비교해서 URI가 일치하는 컨트롤러를 호출
3. 컨트롤러에서 뷰 렌더링 과정을 통해 JSP로 전달
4. JSP에서 HTML을 만들어서 웹 브라우저에 전달

### 1.3 ControllerV1 interface
서블릿과 비슷한 모양의 컨트롤러 인터페이스를 도입하여 프론트 컨트롤러가 해당 인터페이스를 호출함으로써 추상화에 의존하게 만든다. (process)

### 1.4 Controller
비즈니스 로직과 뷰 렌더링을 구현한다.

### 1.5 FrontControllerServletV1
- urlPattern의 마지막에 `/v1/*` 을 함으로써 v1의 하위 경로는 무조건 해당 url을 거쳐야한다.
- Map<url, 컨트롤러 인터페이스>를 통해 컨트롤러의 매핑 정보를 생성 (다형성 이용)
- 요청 메세지에서 URI 정보를 뽑아 매칭 정보와 비교 후 일치하는 controller의 process 메서드 실행 

## 2. v2
View 분리

### 2.1 MyView
각 컨트롤러에 구현되어있는 렌터링 과정을 MyView라는 하나의 객체만든다.

### 2.2 과정
1. 컨트롤러는 해당 url을 MyView 객체에 담아 FrontController에 반환
2. MyView 객체에서 뷰 렌더링 과정을 통해 JSP로 전달
3. JSP에서 HTML을 만들어서 웹 브라우저에 전달

## 3. v3
Model, viewResolver 추가
- 컨트롤러 입장에서는 HttpServletRequest와 HttpServletResponse 객체를 사용하지 않는 컨트롤러도 있기 때문에 꼭 필요한 것은 아니다.
- 따라서 Model 객체를 추가하여 서블릿의 종속성을 없앤다.
- viewResolver를 이용해 논리적 경로를 물리적 경로로 바꾼다.

### 3.1 ModelView
- 서블릿 종속성 제거
- 논리적 경로를 반환함으로써 중복된 경로와 확장자를 제거

### 3.2 과정
1. 요청 메세지의 데이터를 Mapping 하여 컨트롤러에 파라미터로 전달
2. 컨트롤러에서 비즈니스 로직을 수행한 후 view에 필요한 정보들을 ModelView 객체에 HashMap을 이용해 저장한 후 ModelView객체를 반환 (경로, 저장데이터, 저장 목록 등)
3. viewResolver를 통해 url을 조립한 후 MyView 객체에 담아 반환
4. MyView 객체의 render() 메서드에 model을 추가해서 JSP에 전달

## 4. v4
좋은 프레임워크는 실용성이 있어야한다. 지금까지는 매번 ModelView를 생성하고 반환했는데 이부분을 좀 더 실용적이게 바꿔보자.

### 4.1 과정
1. ModelView 대신 FrontController에서 view에 출력할 데이터를 저장할 model HashMap을 생성한다.
2. 컨트롤러를 호출할 때 요청 데이터와 model을 파라미터로 같이 보낸다.
3. 해당 컨트롤러에서 비즈니스 로직을 수행한 후 결과를 model에 담고 viewName(url)만 반환한다.
4. 이하는 v3와 동일

## 5. v5
다양한 방식의 process로 비즈니스 로직을 구현하고 싶을 때

### 5.1 어댑터 패턴
- 핸들러: 컨트롤러의 이름을 더 넓은 범위인 핸들러로 변경 (어댑터가 있기 때문에 컨트롤러의 개념에 국한되지 않고 어떤 것이든 해당하는 종류의 어댑터만 있다면 모두 처리할 수 있다.)
- 핸들러 어댑터: 다양한 종류의 컨트롤러를 호출한다.

### 5.2 과정
1. 초기화 작업: 핸들러들을 url로 Mapping, 핸들러 어댑터들을 리스트에 추가
2. getHandler(): 요청에 맞는 url의 객체를 Mapping 정보에서 꺼낸다.
3. getHandlerAdapter(): 해당 handler에 맞는 어댑터를 꺼낸다. (supports로 비교)
4. handle(): 해당 어댑터에서 handle()을 호출해 handler와 request, response를 인자로 전달해서 ModelView 객체를 반환받는다. (요청 데이터 Mapping, 조회 + process)
    + 다양한 process를 처리할 수 있다.
5. ModelView에서 논리적 경로를 받아서 viewResolver로 물리적 경로 만든 후 MyView 객체에 넣어 반환받는다.
6. view에 필요한 데이터를 ModelView에서 꺼내 request와 response를 인자로 render()메서드를 실행한다.
7. 받은 데이터를 기반으로 JSP에서 HTML을 만들어 웹 브라우저에 전달한다.

### 5.3 MyHandlerAdapter
어댑터용 인터페이스
```java
public interface MyHandlerAdapter {

    boolean supports(Object handler);

    ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException;
}
```
- supports(): 어댑터가 해당 컨트롤러를 처리할 수 있는지 판단하는 메서드
- handle(): 실제 컨트롤러를 호출하고, 그 결과로 ModelView를 반환

### 5.4 ControllerV4HandlerAdapter
실용성 + 어댑터

```java
public class ControllerV4HandlerAdapter implements MyHandlerAdapter {
    @Override
    public boolean supports(Object handler) {
        return (handler instanceof ControllerV4);
    }

    @Override
    public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
        ControllerV4 controller = (ControllerV4) handler;   // Object 객체 handler를 해당 handler로 형변환
        Map<String, String> paramMap = createParamMap(request); // 요청 데이터를 Mapping
        HashMap<String, Object> model = new HashMap<>();    // 비즈니스 로직을 수행 후 view에 필요한 데이터를 담을 HashMap 생성

        String viewName = controller.process(paramMap, model);  // 비즈니스 로직 실행 후 논리적 경로 반환

        ModelView mv = new ModelView(viewName); // 해당 논리적 경로 ModelView 객체를 생성
        mv.setModel(model); // ModelView 객체에 view에 필요한 데이터를 저장

        return mv;
    }

    private static Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();

        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }
}
```

## 6. 애노테이션 기반의 MVC
여기서 애노테이션을 사용해서 컨트롤러를 편리하게 발전시키기 위해선 애노테이션을 지원하는 어댑터를 추가하면 된다. 다형성과 어댑터로 기존 구조를 유지한 상태로 프레임워크의 기능을 확장할 수 있다.

## Reference
> 스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술 [김영한]

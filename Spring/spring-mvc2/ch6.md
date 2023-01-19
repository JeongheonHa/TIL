# 로그인 처리 - 쿠키, 세션

## 1. 로그인 로직 구현

### 1.1 Member
id, loginId, name, password를 갖는 멤버 클래스 생성

### 1.2 MemberRepository
멤버 객체를 저장하고 조회할 수 있는 저장소 생성
- save(), findById(), findByLoginId(), findAll() 구현


### 1.3 MemberController
회원가입 폼 컨트롤러 구현

### 1.4 LoginService
로그인 기능을 수행하는 비즈니스 로직 구현
- login(loginId, password)
```java
Service
@RequiredArgsConstructor
public class LoginService {

    private final MemberRepository memberRepository;

    // null 이면 로그인 실패
    public Member login(String loginId, String password) {
        return memberRepository.findByLoginId(loginId)
                .filter(m -> m.getPassword().equals(password))
                .orElse(null);
    }
}
```

### 1.5 LoginForm
로그인을 수행할 때 쓰일 Form 객체 생성 (loginId, password)

### 1.6 LoginController
로그인 폼 컨트롤러 구현

## 2. 쿠키
쿠키: 클라이언트 로컬에 저장되는 키와 값이 들어있는 작은 데이터 파일
- 쿠키를 통해 로그인 상태를 유지시키고 로그아웃할 수 있는 기능 구현할 수 있다.
- 기본적으로 HTTP 프로토콜은 `connectionless`, `stateless` 구조를 갖기 때문에 서버는 해당 클라이언트가 누구인지 매번 확인해야한다.
- 클라이언트는 해당 경로로 요청을 보낼 때마다 쿠키를 헤더에 포함시켜 요청한다.

### 2.1 쿠키의 종류
- 영속 쿠키: 만료 날짜를 입력하면 해당 날짜까지 유지
- 세션 쿠키: 만료 날짜를 생략하면 브라우저 종료시 까지만 유지

### 2.1 로그인시 세션 쿠키 생성
LoginController의 성공 로직에 추가.
```java
// 쿠키 생성 Cookie(쿠키 이름, 값), 파라미터 둘다 String이다.
Cookie idCookie = new Cookie("memberId", String.valueOf(loginMember.getId()));
// HttpServletResponse 객체에 쿠키를 담는다.
response.addCookie(idCookie);
```

### 2.2 로그인 완료 후 로그인된 홈 화면으로 이동
`@CookieValue(name = "memberId", required = false) Long memberId`: 요청 헤더에 memberId라는 이름의 쿠키 값을 가져온다.
- required = false: 쿠키가 아직 생성되지 않은 사용자 또한 홈 화면으로 이동할 수 있어야한다.
- 쿠키를 저장할 때 값을 String으로 저장했지만 typeconverter로 Long 타입으로 받아온다.
```java
@Controller
@RequiredArgsConstructor
public class HomeController {
    private final MemberRepository memberRepository;

    @GetMapping("/")
    public String homeLogin(@CookieValue(name = "memberId", required = false) Long memberId, Model model) {
        if (memberId == null) { // 로그인 쿠키가 없는 사용자는 home으로 보낸다.
            return "home";
        }

        // 로그인에 성공
        Member loginMember = memberRepository.findById(memberId);

        if (loginMember == null) { // 로그인 쿠키가 있어도 회원이 없으면 home으로 보낸다. (DB에 없을 수 있음. ex. 쿠키가 너무 옛날에 만들어진 경우)
            return "home";
        }
        
        model.addAttribute("member", loginMember);
        return "loginHome";
    }
}
```

### 2.3 로그아웃

#### 로그아웃 하는 방법
1. 세션 쿠키이므로 웹브라우저 종료시 종료한다.
2. 서버에서 해당 쿠키의 종료 날짜를 0으로 지정한다.

```java
// 로그아웃 버튼을 누른 경우
@PostMapping("/logout")
public String logout(HttpServletResponse response) {
    expireCookie(response, "memberId");
    return "redirect:/";
}
private static void expireCookie(HttpServletResponse response, String cookieName) {
    Cookie cookie = new Cookie(cookieName, null);   // 해당 이름의 쿠키 값을 null로 설정
    cookie.setMaxAge(0);    // 쿠키의 종료 날짜를 0으로 지정
    response.addCookie(cookie); // 해당 쿠키를 HttpServletResponse 객체에 추가
}
```

### 2.4 쿠키를 이용한 로그인에서 보안 문제
- 쿠키 값은 임의로 변경할 수 있다.
- 쿠키에 보관된 정보는 훔쳐갈 수 있다.
- 쿠키를 훔쳐가면 평생 사용할 수 있다.

#### 보안 문제 해결
- 쿠키에 중요한 값이 아닌 사용자 별로 예측 불가능한 임의의 토큰을 노출한다. (UUID)
- 서버에서는 토큰과 사용자 id를 매핑해서 인식하고 서버에서 토큰을 관리한다. (session 저장소)
- 토큰이 털려도 시간이 지나면 사용할 수 없도록 해당 토큰의 만료시간을 짧게 유지시킨다.
- 해킹이 의심되면 서버에서 해당 토큰을 강제로 제거한다.

## 3. session
세션: 쿠키를 기반으로하고 있지만, 사용자 정보 파일을 서버에서 관리한다.

### 3.1 session 저장소
서버에서 key는 예상 불가능한 sessionId, value는 회원 객체를 저장하는 session 저장소로 쿠키를 관리힌다.

### 3.2 세션 동작 과정
1. 웹 브라우저에서 로그인을 해서 쿠키 생성
2. 해당 쿠키의 값을 key로 회원 객체를 value로 해서 session 저장소에 저장
3. 서버는 클라이언트에 `쿠키이름 = sessionId`로 해당 쿠키 이름에 sessionId만 응답 헤더에 담아 응답한다.
4. 클라이언트는 자신의 로컬의 쿠키 저장소에 해당 쿠키를 저장
5. 해당 서버로 요청을 보낼 때마다 쿠키를 header에 추가해서 요청
6. 서버는 요청으로 온 쿠키의 sessionId를 세션 저장소에서 확인

### 3.3 세션 직접 만들기
```java
@Component
public class SessionManager {

    public static final String SESSION_COOKIE_NAME = "mySessionId";
    private Map<String, Object> sessionStore = new ConcurrentHashMap<>();

    /**
     * 세션 생성
     * sessionId 생성
     * 세션 저장소에 sessionId에 보관할 값 저장
     * sessionId로 응답 쿠키를 생성해서 클라이언트에 전달
     */

    public void createSession(Object value, HttpServletResponse response) {
        //sessionId 생성, 값을 session 저장소에 저장
        String sessionId = UUID.randomUUID().toString();
        sessionStore.put(sessionId, value);

        //쿠키 생성
        Cookie mySessionCookie = new Cookie(SESSION_COOKIE_NAME, sessionId);
        response.addCookie(mySessionCookie);
    }

    /**
     * 세선 조회
     */
    public Object getSession(HttpServletRequest request) {
        Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);
        if (sessionCookie == null) {
            return null;
        }

        return sessionStore.get(sessionCookie.getValue());
    }

    /**
     * 세션 만료
     */
    public void expire(HttpServletRequest request) {
        Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);
        if (sessionCookie != null) {
            sessionStore.remove(sessionCookie.getValue());
        }
    }

    public Cookie findCookie(HttpServletRequest request, String cookieName) {
        if (request.getCookies() == null) {
            return null;
        }
        return Arrays.stream(request.getCookies())
                .filter(cookie -> cookie.getName().equals(cookieName))
                .findAny()
                .orElse(null);
    }
}
```

### 3.4 세션 적용

#### LoginController
```java
// 로그인
// 세션 관리자 의존성 주입 받기
private final SessionManager sessionManager;
// 세션 관리자를 통해 세션을 생성하고 세션 저장소에 저장
@PostMapping("/login")
public String loginV2(@Validated @ModelAttribute LoginForm form, BindingResult bindingResult, HttpServletResponse response) {
    Member loginMember = loginService.login(form.getLoginId(), form.getPassword());
    sessionManager.createSession(loginMember, response);
    return "redirect:/";
}

// 로그아웃
@PostMapping("/logout")
public String logoutV2(HttpServletRequest request) {
    sessionManager.expire(request);
    return "redirect:/";
}
```

#### HomeController
`@CookieValue`가 아닌 세션 관리자를 통해 로그인 홈 컨트롤러 구현
```java
@GetMapping("/")
public String homeLoginV2(HttpServletRequest request, Model model) {
    //세션 관리자에 저장된 회원 정보 조회
    Member member = (Member) sessionManager.getSession(request);

    if (member == null) {
        return "home";
    }

    model.addAttribute("member", member);
    return "loginHome";
}
```

## 4. HttpSession
HttpSession은 서블릿이 제공하는 SessionManager이다.
- HttpSession 객체를 생성하면 쿠키 이름은 JSESSIONID이고 값은 UUID로 추정 불가능한 값인 쿠키가 생성된다.
- 서버는 `Cookie: JSESSIONID=5B78E23B513F50164D6FDD8C97B0AD05` 형태의 쿠키를 응답 헤더에 말아서 클라이언트에 전달한다.
- JSESSIONID 생성은 톰캣이 담당한다.

### 4.1 HttpSession 구조
<img src = "https://user-images.githubusercontent.com/108064146/213422300-261178e4-c456-49be-838e-ef33bef07d98.png">


### 4.2 HttpSession 처리 과정
1. 서버로 요청이 온다.
2. 요청 메세지에 쿠키가 있으면 해당 쿠키의 값(uuid)을 확인해서 세션 저장소에서 해당 세션을 꺼낸다.
3. 요청 메세지에 쿠키가 없으면 새로운 세션을 생성하고 응답 헤더에 새로운 세션 id(uuid)를 추가한다.
4. 해당 세션에 로그인한 회원 객체를 저장하고 세션 저장소에 저장한다.
5. 요청을 처리하고 응답을 보낸다.
6. 응답을 보낼 때 새로운 세션이 생성된 경우에 쿠키를 말아서 보낸다.

### 4.2 HttpSession 적용

```java
// 
public class SessionConst {
    public static final String LOGIN_MEMBER = "loginMember";
}
```

#### LoginController

```java
// 로그인
@PostMapping("/login")
public String loginV3(@Validated @ModelAttribute LoginForm form, BindingResult bindingResult, HttpServletRequest request) {
    Member loginMember = loginService.login(form.getLoginId(), form.getPassword());
    // 세션이 있으면 있는 세션을 반환, 없으면 신규 세션을 생성
    HttpSession session = request.getSession();
    // 세션에 로그인 회원 정보 보관
    session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);
    return "redirect:/";
}

// 로그아웃
@PostMapping("/logout")
public String logoutV3(HttpServletRequest request) {
    //세션을 삭제한다.
    HttpSession session = request.getSession(false); 
    if (session != null) {
        session.invalidate();
    }
    return "redirect:/";
  }
```

- request.getSession(true): 세션이 있으면 기존 세션 반환, 세션이 없으면 새로운 세션 생성해서 반환 (기본값이라 true 생략 가능)
- request.getSession(false): 세션이 있으면 기존 세션 반환, 세션이 없으면 새로운 세션 생성 x
- session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember): 세션에 데이터 보관 
- session.invalidate(): 세션을 제거한다.

#### HomeController
HttpSession으로 로그인 홈 컨트롤러 구현
```java
@GetMapping("/")
public String homeLoginV3(HttpServletRequest request, Model model) {
    //세션이 없으면 home
    HttpSession session = request.getSession(false); 
    if (session == null) {
        return "home";
    }
    Member loginMember = (Member) session.getAttribute(SessionConst.LOGIN_MEMBER);
    //세션에 회원 데이터가 없으면 home 
    if (loginMember == null) {
        return "home";
    }
    //세션이 유지되면 로그인으로 이동 
    model.addAttribute("member", loginMember); 
    return "loginHome";
}
```

## 5. @SessionAttribute
HttpSession을 편리하게 사용할 수 있도록 스프링이 지원하는 애노테이션이다.
- 세션을 찾고 세션에 들어있는 데이터를 대신 찾아준다.

### 5.1 @SessionAttribute 적용

#### HomeController

```java
@GetMapping("/")
public String homeLoginV3Spring(
        @SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false) Member loginMember,
        Model model) {

    //세션에 회원 데이터가 없으면 home으로 이동
    if (loginMember == null) {
        return "home";
    }

    //세션에 회원 데이터가 이ㅣㅆ으면 loginHome으로 이동
    model.addAttribute("member", loginMember);
    return "loginHome";
}
```

## 6. TrackingModes
웹 브라우저가 쿠키를 지원하지 않을 때 쿠키 대신 URL을 통해서 세션을 유지시키는 방법
- HTTP 프로토콜은 connectionless 이기 때문에 서버 입장에서는 웹 브라우저가 쿠키를 지원하는지 하지 않는지 모르기 때문에 헤더에 쿠키를 추가하는 방법과 URL에 추가해서 보내는 방법 2가지를 모두 보내는 것이다. 
- 하지만, 사용하지 않기 때문에 application.properties에서 `server.servlet.session.tracking-modes=cookie`를 등록해서 꺼두자.

## 7. HttpSession이 제공하는 정보
- `sessionId` : 세션Id, JSESSIONID 의 값이다.
- `maxInactiveInterval` : 세션의 유효 시간, 예) 1800초, (30분)
- `creationTime` : 세션 생성일시
- `lastAccessedTime` :세션과 연결된 사용자가 최근에 서버에 접근한 시간, 클라이언트에서 서버로 sessionId(JSESSIONID)를 요청한 경우에 갱신된다.
- `isNew` : 새로 생성된 세션인지, 아니면 이미 과거에 만들어졌고, 클라이언트에서 서버로 sessionId(JSESSIONID)를 요청해서 조회된 세션인지 여부

### 7.1 세션 타임아웃
로그아웃을 누를 경우 session.invalidate()가 호출되어 세션이 제거되지만, 대부분은 그냥 웹 브라우저를 종료해버리기 때문에 connectionless에 따라 서버는 해당 사용자가 웹 브라우저를 종료했는지 알 수 없다.

#### 세션이 세션 저장소에 누적될 경우 문제점
- 오랜 기간 세션 저장소에 해당 sessionid의 세션이 존재할 경우 해킹의 위험이 있다.
- 세션은 기본적으로 메모리에 생성되며 메모리는 한정적이다.

#### 세션 타임아웃 설정
HttpSession은 사용자가 최근에 서버에 요청을 보낸 시간을 기준으로 30분 정도를 유지해주며 30분이 지나면 자동으로 세션이 제거된다.
- 글로벌 설정: application.properties에서 `server.servlet.session.timeout=60` (글로벌의 경우 분 단위로)
- 특정 세션 설정: 컨트롤러에서 `session.setMaxInactiveInterval(1800);` 

#### 세션 타임아웃 발생
lastAccessedTime 이후로 세션 유효시간이 지나면 WAS가 내부에서 해당 세션을 제거

## Reference
> 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술 [김영한]

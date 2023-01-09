# Thymeleaf 기능

## 1. 타임리프 특징
- 서버 사이드 렌더링
- 네츄럴 템플릿
- 스프링 통합 지원

### 1.1 서버 사이드 렌더링 (SSR)
타임리프느 백엔드 서버에서 HTML을 동적으로 렌더링하는 용도로 사용된다.

### 1.2 네츄럴 템플릿
해당 파일을 그래로 웹 브라우저에서 열어도 정상적인 HTML을 확인할 수 있고 렌더링 후에도 렌더링이 적용된 HTML 소스 코드를 확인할 수 있다.

### 1.3 스프링 통합 지원
스프링의 다양한 기능을 편리하게 사용할 수 있도록 지원한다.

## 2. 타임리프 기능

### 2.1 타임리프 선언
```html
<html xmlns:th="http://www.thymeleaf.org">
```

### 2.2 텍스트
- `text`, `utext`
- `[[...]]`, `[(...)]`

```html
<li>th:text 사용 <span th:text="${data}"></span></li> 
<li>컨텐츠 안에서 직접 출력하기 = [[${data}]]</li>

// unescape: escape 적용을 풀어서 HTML 엔티티로 변경하지 않고 태그로 인식할 수 도록한다.
<li>th:utext = <span th:utext="${data}"></span></li>
<li><span th:inline="none">[(...)] = </span>[(${data})]</li>
```

#### HTML 엔티티
웹 브라우저는 "<"를 HTML 태그의 시작으로 인식하기 때문에 "<"를 문자로 인식할 수 있도록 하는 "&~;"형식으로 변경한 문자를 `HTML 엔티티`라고 하고 이렇게 변경하는 것을 `escape`라고 한다.
- 타임리프가 제공하는 텍스트 기능은 기본적으로 escape가 적용되어있다.

#### th:inline="none"
해당 태그 안에서는 타임리프가 해석하지 말라는 옵션 위의 경우는 <span> </span>

### 2.3 변수 표현식
- `${...}`
- 모델에 저장되어 있는 key로 접근

#### 객체
- user.username
- user['username']
- user.getUsername()

#### List
- user[0]으로 객체를 얻을 수 있다.

#### Map
- userMap['key']로 객체를 얻을 수 있다.

### 2.4 지역 변수 선언
- `th:with`
- first를 지역 변수로 지정한다. scope는 `<div> </div`>가 된다.
```html
<div th:with="first=${users[0]}">
<p>처음 사람의 이름은 <span th:text="${first.username}"></span></p> </div>
```

### 2.5 기본 객체 지원
locale을 제외한 나머지는 스프링 부트 3.0부터는 지원하지 않는다.
- ${#request}
- ${#response}
- ${#session} 
- ${#servletContext}
- ${#locale}

### 2.6 편의 객체

#### HTTP 요청 파라미터 접근
- ${param.paramData}

#### HTTP 세션 접근
- ${session.sessionData}

#### 스프링 빈 접근
- ${@helloBean.hello('Spring!')}

### 2.7 유틸리티 객체와 날짜
외우는 것보다 해당 기능이 필요할 때 찾아서 사용하도록 하자.
- #message : 메시지, 국제화 처리
- #uris : URI 이스케이프 지원
- #dates : java.util.Date 서식 지원 
- #calendars : java.util.Calendar 서식 지원 
- #temporals : 자바8 날짜 서식 지원
- #numbers : 숫자 서식 지원
- #strings : 문자 관련 편의 기능
- #objects : 객체 관련 기능 제공
- #bools : boolean 관련 기능 제공
- #arrays : 배열 관련 기능 제공
- #lists , #sets , #maps : 컬렉션 관련 기능 제공 
- #ids : 아이디 처리 관련 기능 제공, 뒤에서 설명

> 유틸리티 객체 설명: <https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#expression-utility- objects>
> 예시: <https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#appendix-b-expression- utility-objects>

#### 날짜
자바 8 날짜 표현
```html
// 자주 사용하는 표현
<li>yyyy-MM-dd HH:mm:ss = <span th:text="${#temporals.format(localDateTime,yyyy-MM-dd HH:mm:ss')}"></span></li>
```
- 나머지는 필요할 때 찾아보자.

### 2.8 URL 링크
- `@{...}`
- `()`안에 경로 변수와 쿼리 파마미터 값을 지정할 수 있다. 경로 변수에 들어갈 수 없는 부분은 쿼리 파라미터로 넘어간다.
- 절대 경로, 상대 경로, 프로토콜 기준을 표현할 수 도 있다.

#### 단순 URL
```html
@{/hello}

/hello
```
#### 경로 변수 + 쿼리 파라미터
```html
@{/hello/{param1}(param1=${param1}, param2=${param2})}

/hello/data1?param2=data2
```

### 2.9 리터럴
타임리프에는 문자, 숫자, boolean, null 리터럴이있으며 해당 리터럴들은 연산을 통해 
- 타임 리프에서 문자 리터럴은 항상 `''`으로 감싸야 한다.
- `A-Z`,` a-z`,`0-9`, `[]`, `.`, `-`, `_` : 해당 문자로 이어진 경우에는 1개의 토큰으로 봐서 `''`로 감싸지 않아도 된다.

#### 리터럴 대체
- `|...|`
- 리터럴 대체 문법을 사용하면 연산을 하지않고 표현할 수 있다.

```html
<span th:text="|hello ${data}|">
```

### 2.10 연산
- `+`, `-`와 같은 산술 연산: 자바와 동일
- 비교연산: HTML 엔티티를 사용해야 하는 부분을 주의하자
    + `>`(gt), `<`(lt), `>=`(ge), `<=`(le), `!`(not), `==`(eq), `!=`(neq, ne)
- 조건식: 자바의 조건식과 유사하다.
    + `(10 % 2 == 0)? '짝수':'홀수'`
- Elvis 연산자: 조건식의 편의 버전으로 데이터가 없는 경우에 `:`다음 부분이 출력된다.
    + `${data}?: '데이터가 없습니다.'`
- No-Operation: `_` 인 경우 마치 타임리프가 실행되지 않는 것 처럼 동작한다.
    + `<li>${data}?: _ = <span th:text="${data}?: _">데이터가 없습니다.</span></li>`

### 2.11 속성 값 설정
- th:xx로 속성을 적용하면 기존 속성을 대체하고 기존 속성이 없으면 새로 만든다.

#### 속성 값 설정
- th:name="userA"
#### 속성 값 추가
- th:attrappend : 속성 값의 뒤에 값을 추가한다. 
- th:attrprepend : 속성 값의 앞에 값을 추가한다. 
- th:classappend : class 속성에 자연스럽게 추가한다.

#### checked 처리
- th:checked = true: 체크
- th:checked = false: checked 속성 자체를 제거

### 2.11 반복
- th:each
- 배열, list, map 등 java.util.Iterable, java.util.Enumeration 을 구현한 모든 객체에 가능
```html
<tr th:each="user, userStat : ${users}">
    <td th:text="${userStat.count}">username</td>
    <td th:text="${user.username}">username</td>
    <td th:text="${user.age}">0</td>
</tr>
```

#### 반복 상태 유지
- 반복의 두 번째 파라미터로 반복의 상태를 확인 할 수 있다.
- 생략하면 변수명(user) + Stat가 상태 확인 파라미터가 된다.
- 반복 상태 유지 기능
- index : 0부터 시작하는 값
- count : 1부터 시작하는 값
- size : 전체 사이즈
- even , odd : 홀수, 짝수 여부( boolean ) 
- first , last :처음, 마지막 여부( boolean ) 
- current : 현재 객체

### 2.12 조건부 평가
조건이 맞지 않으면 태그 자체를 렌더링하지 않는다. 조건이 맞으면 렌더링
```html
// if, unless
<span th:text="'미성년자'" th:if="${user.age lt 20}"></span>
<span th:text="'미성년자'" th:unless="${user.age ge 20}"></span>

//switch
<td th:switch="${user.age}">
    <span th:case="10">10살</span> 
    <span th:case="20">20살</span> 
    <span th:case="*">기타</span>   // *: default
</td>
```

### 2.13 주석

#### 표준 HTML 주석 (잘 사용x)
- <!--     -->
- html, 타임리프 모두 주석으로 남겨놓는다.

#### 타임리프 파서 주석 (많이 사용)
- <!--/*     */-->
- 타임리프가 렌더링에서 해당 주석 부분을 제거한다.

#### 타임리프 프로토타입 주석 (사용x)
- <!--/*/     /*/-->
- 타임리프로 렌더링된 경우에만 렌더링되고 렌더링되지 않은 전체 경로에서는 주석처리한다.

### 2.14 블록
- `<th:block>`: 1개의 범위가 아닌 div를 동시에 처리해서 루프를 돌릴 경우 사용
- HTML 태그가 아닌 타임리프의 유일한 자체 태그이다.
- th:each로 해결할 수 없는 경우에만 사용한다.

### 2.15 자바스크립트 인라인
```html
<script th:inline="javascript">
    // 텍스트 렌더링
    var username = [[${user.username}]];
    var age = [[${user.age}]];

    //자바스크립트 내추럴 템플릿
    var username2 = /*[[${user.username}]]*/ "test username";

    //객체
    var user = [[${user}]];

</script>
```

#### 자바스크립트 인라인 each
```html
<script th:inline="javascript">
    [# th:each="user, stat : ${users}"]
    var user[[${stat.count}]] = [[${user}]];
    [/]
</script>
```

### 2.16 템플릿 조각
- th:insert : 현재 태그 내부에 추가
- th:replace : 현재 태그를 완전히 대체
```html
// 템플릿 조각
<footer th:fragment="copyParam (param1, param2)">
    <p>파라미터 자리 입니다.</p>
    <p th:text="${param1}"></p> 
    <p th:text="${param2}"></p>
</footer>
```
```html
// 템플릿 조각 가져와 사용 (경로 :: fragment)
<div th:replace="~{template/fragment/footer :: copyParam ('데이터1', '데이터 2')}"></div>
```

### 2.17 템플릿 레이아웃
- 템플릿 경로 :: 템플릿에 바꿀 태그
- 태그 자체가 넘어가 해당 레이아웃의 ${태그}에 치환시킨 header를 Main에 가져와서 렌더링한다.
```html
<head th:replace="template/layout/base :: common_header(~{::title},~{::link})">
```
```html
// layout 템플릿
<head th:fragment="common_header(title,links)">
```

### 2.18 템플릿 레이아웃 확장
- html에 적용함으로써 태그를 해당 레이아웃의 ${태그}에 치환시킨 html을 Main에 가져와서 렌더링한다.
```html
<html th:replace="~{template/layoutExtend/layoutFile :: layout(~{::title},~{::section})}"
        xmlns:th="http://www.thymeleaf.org">
```
```html
<html th:fragment="layout (title, content)" xmlns:th="http://
  www.thymeleaf.org">
```

## Reference
> 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술 [김영한]

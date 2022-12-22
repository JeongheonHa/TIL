# HTTP 헤더

## 1. HTTP 헤더의 용도
- HTTP 전송에 필요한 모든 부가 정보
- 필요시 임의의 헤더 추가 가능

## 2. 표현
- Content-Type: 표현 데이터의 형식 (html인지, json인지 ...)
- Content-Encoding: 표현 데이터의 압축 방식 (gzip인지, delfate인지, identity인지 ...)
- Content-Language: 표현 데이터의 자연 언어 (ko, en, en-US ...)
- Content-Length: 표현 데이터의 길이 (바이트 단위)

## 3. 협상(Content Negotiation)
클라이언트가 선호하는 표현 요청 (협상 헤더는 요청시에만 사용)
- Accept: 클라이언트가 선호하는 미디어 타입 전달
- Accept-Charset: 클라이언트가 선호하는 문자 인코딩
- Accept-Encoding: 클라이언트가 선호하는 압축 인코딩
- Accept-Language: 클라이언트가 선호하는 자연 언어

### 협상과 우선순위
- Quality Values(q)값 사용

#### 높은 우선순위
- 0 ~ 1, 클수록 높은 우선순위를 갖는다.
- 생략하면 1 이다.
- 예시. `Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7`

#### 구체적인 것이 우선
- `Accept: text/*, text/plain, text/plain;format=flowed, */*`
- `text/plain;format=flowed` : 1
- `text/plain` : 2
- `text/*` : 3
- `*/*` : 4

#### 구체적인 것을 기준으로 미디어 타입을 맞춘다.

## 4. 전송 방식
- 단순 전송
- 압축 전송
- 분할 전송 (Content-Length 보내면 안됨)
- 범위 전송

## 5. 일반 정보
- Form: 유저 에이전트의 이메일 정보 (요청)
- Referer: 이전 웹 페이지 주소 (요청)
- User-Agent: 유저 에이전트 애플리케이션 정보 (요청)
- Server: 요청을 처리하는 오리진 서버의 소프트웨어 정보 (응답)
- Date: 메세지가 생성된 날짜 (응답)

## 6. 특별한 정보
- Host: 요청한 호스트 정보 (도메인, 필수)
- Location: 페이지 리다이렉션
- Allow: 허용 가능한 HTTP 메서드
- Retry-After: 유저 에이전트가 다음 요청을 하기까지 기다려야 하는 시간

## 7 인증
- Authorization: 클라이언트 인증 정보를 서버에 전달
- WWW-Authenticate: 리소스 접근시 필요한 인증 방법 정의 (401 Unauthorized응답과 함께 사용)

## 8. 쿠키
- Set-Cookie: 서버에서 클라이언트로 쿠키 전달(응답)
- Cookie: 클라이언트가 서버에서 받은 쿠키를 저장하고, HTTP 요청시 서버로 전달

### 과정
1. 웹 브라우저가 로그인한다.
2. 서버에서 유저 정보를 쿠키에 말아서 응답
3. 웹 브라우저 내부의 쿠키 저장소에 유저 정보를 저장한다.
4. 로그인 이후 웹 브라우저가 해당 서버에 요청을 보낼 때 마다 쿠키 저장소에서 쿠키 값을 꺼내서 요청에 Cookie라는 http 헤더를 만들어서 요청

### 예시
```html
set-cookie: sessionId=abcde1234; expires=Sat, 26-Dec-2020 00:00:00 GMT; path=/; domain=.google.com; Secure
```

### 사용처
- 사용자 로그인 세션 관리
- 광고 정보 트래킹

### 쿠키정보는 항상 서버에 전송됨
쿠키 정보는 항상 서버에 전송되기 때문에 네트워크 트래픽을 추가 유발시키며 세션 id, 인증 토큰 같은 최소한의 정보만 사용해야한다. 만약 서버에 전송하지 않고, 웹 브라우저 내부에 데이터를 저장하고 싶으면 웹 스토리지 참고하도록 하자.

> ##### 보안에 민감한 데이터는 절대 저장하면 안된다.

### 쿠키의 생명주기
- Set-Cookie: expires=Sat, 26-Dec-2020 04:39:21 GMT
    - 만료일이 되면 쿠키 삭제
- Set-Cookie: max-age=3600 (3600초)
    - 0이나 음수를 지정하면 쿠키 삭제

### 쿠키의 종류
- 세션 쿠키: 만료 날짜를 생략하면 브라우저 종료시 까지만 유지 (웹 브라우저 종료시 로그인 다시하는 경우)
- 영속 쿠키: 만료 날짜를 입력하면 해당 날짜까지 유지

### 쿠키의 도메인
쿠키에 도메인을 지정해서 요청하는 경우 <br>
도메인: domain=example.org

- 도메인을 명시하는 경우
    - domain=example.org를 지정해서 쿠키를 생성한 경우
    - 명시한 문서 기준 도메인 + 서브 도메인을 포함한 도메인에 쿠키가 같이 접근할 수 있다.
    - example.org + dev.example.org

- 도메인을 생략한 경우
    - example.org에서 쿠키를 생성했지만 도메인을 지정하지 않은 경우
    - 현재 문서 기준 도메인만 적용
    - example.org (o), dev.example.org (x)

### 쿠키의 경로
해당 경로를 포함한 하위 경로 페이지만 쿠키 접근 <br>
경로: path=/home
- /home (가능)
- /home/level1 (가능)
- /home/level1/level2 (가능)
- /hello (불가능)

### 쿠키의 보완
- Secure
    + 쿠키는 http, https를 구분하지 않고 전송하지만
    + Secure를 적용하면 https인 경우에만 전송한다.
- HttpOnly
    + XSS 공격 방지
    + 원래 자바스크립트에서 접근이 가능하지만 자바스크립트에서 접근하지 못하도록 만든다.(document.cookie)
    + 대신, HTTP 전송에만 사용할 수 있게 만든다.
- SameSite
    + XSRF 공격 방지
    + 요청 도메인과 쿠키에 설정된 도메인이 같은 경우만 쿠키 전송

## 10. 검증 헤더와 조건부 요청

### 검증 헤더
캐시 데이터와 서버 데이터가 같은지 검증하는 데이터
- `Last-Modified`, `ETag`

### 조건부 요청 헤더
검증 헤더로 조건에 따른 분기
- `If-Modified-Since: Last-Modified` 사용
- `If-None-Match: ETag` 사용
- 조건이 만족하면 200 OK, 조건이 만족하지 않으면 304 Not Modified

### 예시
상황 : 데이터의 캐시 만료 시간이 다 되어서 서버에게 같은 데이터를 요청하는 경우 <br>
캐시 : `cache-control: max-age=60` (서버) <br>
검증 헤더 : `Last-Modified: 2020년 11월 10일 10:00:00` (서버) <br>
조건부 요청 : `if-modified-since: 2020년 11월 10일 10:00:00` (클라)
- 서버에서 기존 데이터를 변경한 경우
    + 다시 변경된 데이터를 클라이언트에게 응답
- 서버에서 기존 데이터를 변경하지 않은 경우
    + 304 Not Modified + 헤더 메타 정보만 응답 (바디 x)
    + 클라이언트는 서버가 보낸 응답 헤더 정보로 캐시의 메타 정보를 갱신
    + 클라이언트는 캐시에 저장되어 있는 데이터 재활용
    + 결과적으로 네트워크 다운로드가 발생하지만 용량이 적은 헤더 정보만 다운로드

### 특징
- 캐시는 단위가 초 단위이기 때문에 1초 미만의 단위로 캐시를 조정하는 것은 불가능하다. (하지만 초 단위가 훨씬 유연하기 떄문에 초 단위를 사용한다.)
- 날짜 기반의 정해진 로직을 사용한다.
- A -> B -> A로 수정한다면 날짜가 다르지만 결과가 같은 데이터로 수정한 것이므로 수정된 데이터들 모두 다운로드한다.

### ETag (Entitiy Tag)
서버에서 별도의 캐시 로직을 관리하는 방법 <br>
캐시용 데이터에 임의의 고유한 버전 이름을 달아서 데이터가 변경되면 해당 이름을 바꾸어서 변경한다. (Hash를 다시 생성) 그 후 ETag만 보내서 같으면 유지하고, 다르면 다시 받는다.
- ETag: "v1.0" (이름은 아무거나 가능)
- ETag: "v1.0" -> "v2.0"
- 클라이언트는 단순히 이 값을 서버에 제공할 뿐 캐시의 메커니즘을 알 수 없다.

## 11. 캐시 제어 헤더
- Cache-Control: 캐시 제어 (중요)
- Pragma: 캐시 제어 (하위 호환)
- Expires: 캐시 유효 기간 (하위 호환)

### Cache-Control
- Cache-Control: max-age
    + 캐시 유효 시간, 초 단위
- Cache-Control: no-cache
    + 데이터는 캐시해도 되지만, 항상 원(origin)서버에 검증하고 사용하라는 뜻
- Cache-Control: no-store
    + 데이터에 민감한 정보가 있으므로 저장하면 안되고 메모리에서 사용하고 최대한 빨리 삭제하라는 뜻
- Cache-Control: public
    + 응답이 public 캐시에 저장되어도 된다는 뜻
- Cache-Control: private
    + 응답이 해당 사용자만을 위한 것으로 private 캐시에 저장되어야 한다는 뜻(기본값)
- Cache-Control: s-maxage
    + 프록시 캐시에만 적용되는 max-age
- Age: 60 (HTTP 헤더)
    + 오리진 서버에서 응답 후 포록시 캐시 내에 머문 시간(초)을 요청

### Pragma
- Pragma: no-cache

### Expires
- 캐시 만료일을 지정할 수 있지만 초 단위로 하는 것이 훨씬 유연함으로 Cache-Control: max-age를 사용하는 것을 권장한다.
- HTTP 1.0 부터 사용 가능
- max-age를 사용하면 Expires는 무시된다.

## 캐시 무효화
문제 : 캐시를 지정하지 않아도 웹 브라우저가 임의로 캐시를 지정할 수 있다. <br>
해결 : 캐시가 되면 안되는 데이터에 캐시 무효화 응답을 보낸다.

### 확실하게 캐시를 무효화 하는 방법
서버에서 http 응답 메세지를 만들 때 아래의 헤더를 모두 넣어줘야한다.
- Cache-Control: no-cache, no-store, must-revalidate
- Pragma: no-cache
    + 과거 HTTP 1.0 에서 요청이 올 수 있으므로 하위 호환해서 작용해야한다. 

### Cache-Control: must-revalidate
- 캐시 만료 후 최초 조회시 원 서버에 검증해야한다.
- 원 서버 접근 실패시 반드시 오류가 발생해야한다. - 504(Gateway Timeout)
- must-revalidate는 캐시 유효 시간이라면 캐시를 사용한다.

### no-cache vs must-revalidate
상황: 프록시 서버와 원 서버간의 네트워크가 순간 단절이 된 경우
- no-cache로 원 서버에 요청한 경우 프록시 서버를 거쳐 원 서버로 갈 수 없으므로 프로시 서버에서 과거의 데이터를 클라이언트에게 응답한다.
- must-revalidate로 원 서버에 요청한 경우 프록시 서버를 거쳐 원 서버로 갈 수 없으므로 프록시 서버에서 504 에러 메세지를 응답한다.

## Reference
- 모든 개발자를 위한 HTTP 웹 기본 지식 [김영한]

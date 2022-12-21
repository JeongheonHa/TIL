# HTTP (HyperText Transfer Protocol)

## HTTP 특징
- 클라이언트 서버 구조
- 무상태 프로토콜
- 비연결성
- HTTP 메세지
- 단순함, 확장 가능

### 클라이언트 서버 구조
- Request Response 구조
- 클라이언트는 서버에 요청을 보내고, 응답을 기다린다.
- 서버가 요청에 대한 결과를 만들어서 응답한다.
- 클라이언트와 서버를 분리하는 것이 중요한 이유는 비지니스 로직과 데이터 같은 것들은 모두 다 서버에 집중하고 클라이언트는 UI, 사용성에 집중해서 관리할 수 있다.

### 무상태 프로토콜 (Stateless)
서버가 클라이언트의 상태(이전 문맥)를 보존하지 않는다는 의미이다.
- 중간에 서버가 바뀌어도 알 수 있도록 클라이언트가 알고있는 상태 + 요청을 서버에게 전달하면 서버는 상태를 보존하지 않고 응답만 한다.
- 무상태는 응답 서버를 쉽게 바꿀 수 있다. (무한한 서버 증설 가능, 트레픽 완화)
- `스케일 아웃` (수평 확장 유리, 위와 같은 말)
- 반대의 의미로 Stateful은 서버가 클라이언트의 상태를 보존한다는 의미이다.
    + 서버가 중간에 장애가 나면 클라이언트는 처음부터 다시 요청을 보내야한다. (하나의 서버만 상태를 유지하고 있으므로)

### 무상태 프로토콜 한계
- 모든 것을 무상태로 설계 할 수는 없다.
- 데이터를 너무 많이 보낸다.
- 무상태 예시 : 로그인이 필요없는 단순한 서비스 소개 화면
- 상태 유지 예시 : 로그인
    + 로그인 한 사용자의 경우 로그인 했다는 상태를 서버에 유지해야한다.
    + 일반적으로 브라우저 쿠키와 세션 등을 사용해서 상태를 유지시킨다.
- 따라서, 대부분의 경우 무상태로 설계하고 꼭 필요한 경우에만 상태 유지로 설계한다.

### 비연결성
- TCP/IP 연결 후 요청과 응답이 끝나면 연결을 끊는다. 
- 일반적으로 초 단위 이하의 빠른 속도로 응답하기 때문에 실제로 요청은 얼마 되지 않는다.
- 이렇게 함으로써 최소한의 자원으로 서버를 유지할 수 있다.

### 비연결성 한계
- 한계 
    + TCP/IP 연결을 새로 맺어야한다. (3 way handshake의 시간 추가)
    + 예를 들면 검색 후 다른 페이지로 넘어가면 새로 연결해야한다.

- 극복
    + 현재는 HTTP 지속 연결(Persistent Connections)로 문제를 해결한다. (http2, http3에서는 더욱 최적화 되어있다.)
    + 지속 연결 : html -> js -> 이미지 응답 등 각각의 데이터의 응답을 모두 받을 때까지 연결을 유지하는 것을 말한다.

### HTTP 메세지

- HTTP 메세지 구조
<img src = "https://user-images.githubusercontent.com/108064146/208590431-e2070aac-e9b3-4777-a022-a02b7ec767c6.JPG">

- HTTP 요청 메세지 / HTTP 응답 메세지 <br>
요청 메세지도 message body를 가질 수 있다.
<img src = "https://user-images.githubusercontent.com/108064146/208590571-c3043ded-79df-4380-acdf-c7be420e56a7.JPG">

#### 시작 라인
request-line 과 status-line이 있다.
#### request-line (요청 메세지)

```html 
GET /search?q=hello&hl=ko HTTP/1.1
```

method SP request-target(path) SP HTTP-version CRLF (SP:공백, CRLF(엔터))
+ HTTP 메서드
    + 서버가 수행해야 할 동작 지정
    + 종류 : GET, POST, PUT, DELETE ...
    + GET : 리소스 조회, POST : 요청 내역 처리
+ 요청 대상
    + absolute-path[?query] (절대경로[?쿼리])
    + 절대경로 : `/`로 시작하는 경로
+ HTTP version

#### status-line (응답 메세지)

```html
HTTP/1.1 200 OK
```

HTTP-version SP status-code SP reason-phrase CRLF
- HTTP 버전
- HTTP 상태 코드
    + 200: 요청 성공
    + 400: 클라이언트 요청 오류
    + 500: 서버 내부 오류
- 이유 문구
    + 사람이 이해할 수 있는 상태 코드 설명

#### HTTP 헤더
```html
요청 메세지
Host: www.google.com

응답 메세지
Content-Type: text/html;charset=UTF-8
Content-Length: 3423
```
field-name ":" OWS field-value OWS (OWS: 띄어쓰기 허용)
- field-name은 대소문자 구분이 없다.
- HTTP 전송에 필요한 모든 부가정보가 다 들어있다. (즉, 메세지 바디를 제외한 필요한 메타 데이터가 모두 들어있다.)
- 필요시 임의의 헤더 추가 가능 (약속된 클라-서버의 경우)

#### HTTP 메세지 바디
```html
<html>
    <body>...<body>
</html>
```
- HTTP 스펙 상에는 GET도 메세지 바디를 작성하여 보낼 수 있지만 아직은 지원하지 않는 서버도 많이 있기 때문에 GET 방식에서는 메세지 바디를 사용하지 않는다.
실제 전송할 데이터
- HTML 문서, 이미지, 영상, JSON 등 
- byte로 표현할 수 있는 모든 데이터 전송 가능

## HTTP 메서드

### API URI 고민
회원 등록, 회원 조회, 회원 수정, 회원 삭제 URI를 설계할 때는 CRUD가 아닌 리소스 식별을 기준으로 설계해야한다.
- 회원이라는 리소스를 URI에 매핑
- 행위에 관한 모든 URI: `/members/{id}`
- URI는 리소스만 식별하고 리소스와 해당 리소스를 대상으로 하는 행위를 분리
    + 리소스: 회원
    + 행위: CRUD
- HTTP 메세지가 대신 행위를 지정해준다.

### HTTP 메서드 종류
- GET: 리소스 조회
- POST: 요청 데이터 처리, 주로 등록에 사용
- PUT: 리소스가 있으면 완전히 대체, 없으면 생성 (덮어쓰기)
- PATCH: 리소스 부분 변경
- DELETE: 리소스 삭제
<br>

기타 메서드 (참고)

- HEAD: GET과 동일하지만 메세지 부분을 제외하고 반환
- OPTIONS: 대상 리소스에 대한 통신 가능 옵션(메서드)을 설명(주로 CORS에서 사용)
- CONNECT: 대상 자원으로 식별되는 서버에 대한 터널을 설정
- TRACE: 대상 리소스에 대한 경로를 따라 메세지 루프백 테스트를 수행

#### GET
리소스 조회
```html
Get /search?q=hello&hl=ko HTTP/1.1
Host: www.google.com
```
- 서버에 전달하고 싶은 데이터는 query(쿼리 파라미터, 쿼리 스트링)를 통해서 전달
- 메세지 바디를 사용해서 데이터를 전달할 수 있지만, 지원하지 않는 곳이 많아서 권장하지 않음

#### POST
요청 데이터 처리, 주로 등록에 사용
```html
POST /members HTTP/1.1
Content-Type: application/json

{
    "username": "hello",
    "age": 20
}
```
- 메세지 바디를 통해 서버로 나의 요청 데이터를 전달한다.
- 서버는 요청 데이터를 처리한다.
    + 서버가 아직 식별하지 않은 `신규 리소스 등록`
    + 데이터 생성/변경을 넘어 `프로세스를 처리` 해야하는 경우
    + POST의 결과로 신규 리소스가 생성되지 않을 수 있음
    + 리소스 식별 만으로 요청 데이터를 처리할 수 없을 경우 `컨트롤 URI` 를 사용한다.
    + 컨트롤 URI: POST /orders/{orderId}/start-delivery
+ 다른 메서드로 처리하기 애매한 경우
    + 예를 들면 JSON으로 조회 데이터를 넘겨야 하는데, GET 메서드를 사용하기 어려운 경우
    + 애매하면 POST를 사용 (POST는 메세지로 오는 모든 데이터를 처리가 가능하지만 기존에 있는 메서드를 사용하는 것이 유리하다.)
- 리소스를 추가하는 것만이 아닌 리소스 URL에 POST 요청이 오면 요청 데이터를 어떻게 처리할 것인지 리소스마다 따로 정해두어야한다.

#### PUT
리소스가 있으면 완전히 대체, 없으면 생성 (기존의 리소스를 모두 지우고 덮어쓰기)

```html
PUT /members/100 HTTP/1.1
Content-Type: application/json

{
    "age": 50
}
```

- 클라이언트가 리소스의 전체 경로를 알고 URI로 지정 (POST와 차이)

#### PATCH
리소스 부분 변경

```html
PATCH /members/100 HTTP/1.1
Content-Type: application/json

{
    "age": 50
}
```

#### DELETE
리소스 제거
```html
DELETE /members/100 HTTP/1.1
Host: localhost:8080
```

### HTTP 메서드의 속성
- `안전` (Safe Methods) : 호출해도 리소스를 변경하지 않아도 되는지 여부
- `멱등` (Idempotent Methods) : 한번 호출하든 100번 호출하든 결과가 똑같은지 여부
    + GET: 몇 번을 조회하든 같은 결과가 조회된다.
    + PUT: 몇 번을 대체하든 같은 결과로 대체된다.
    + DELETE: 몇 번을 제거하든 제거된 상태로 남아있는 것은 같다.
    + POST (멱등 X): 2번 호출하면 같은 결제가 2번 발생한다.
    + 자동 복구 메커니즘에 활용 가능 (서버에서 호출의 결과가 없을 경우 판단을 위해 반복 호출 가능)
    + 외부 요인으로 중간에 리소스가 변경되는 경우는 고려하지 않는다.
- `캐시 가능` (Cacheable Methods) : 응답 결과 리소스를 캐시에서 사용할 수 있는지 여부
    + GET, HEAD, POST, PATCH : 캐시 가능
    + 실제로는 GET, HEAD 정도만 캐시로 사용
    + 캐시는 같은 리소스라는 키가 일치해야하는데 POST, PATCH는 본문 내용까지 캐시 키로 고려해야하기 때문에 구현이 쉽지않다.
    + GET은 URI만 키로 잡고 캐시를 구현하면 되기 때문에 구현이 쉽다.

<img src = "https://user-images.githubusercontent.com/108064146/208832303-5eea7588-4976-4686-a932-fe41be99c428.png">

> ###### 출처 : https://ko.wikipedia.org/wiki/HTTP

### HTTP 메서드 활용
상황 : 클라이언트에서 서버로 데이터를 전송

#### 데이터의 전달 방식
1. 쿼리 파라미터를 통한 전송
    + GET
    + 주로 정렬 필터(검색어)에 사용

2. 메세지 바디를 통한 데이터 전송
    + POST, PUT, PATCH
    + 회원 가입, 상품 주문, 리소스 등록, 리소스 변경

#### 1. 정적 데이터 조회의 경우
쿼리의 파라미터 없이 단순 리소스 경로만으로 조회 가능
- 이미지, 정적 텍스트 문서

#### 2. 동적 데이터 조회의 경우
쿼리 파라미터를 사용하여 데이터를 조회
- 쿼리 파리미터를 이용하여 서버에서 key-value로 데이터를 꺼내서 응답
- 검색, 게시판 목록에서 정렬 필터(검색어)
- 조회 조건을 줄여주는 필터, 조회 결과를 정렬하는 정렬 조건에 주로 사용된다.

#### 3. HTML Form을 통한 데이터 전송의 경우

<img src="https://user-images.githubusercontent.com/108064146/208842263-a80c3fe9-67ca-4115-b397-9522f3944d07.png">

- POST 방식으로 "/save"라는 URI에 저장한다고 가정할 때
- form의 submit 버튼을 누르면 오른쪽과 같은 http 메세지를 생성하여 서버에 전달한다.
- Content-Type: 전송되는 데이터의 타입을 명시하기 위해 header에 실리는 정보이다.
- `application/x...` : default값이 해당 형식이며 아래와 같은 타입을 말하는 클라이언트와 서버간의 약속을 말한다.
    + form의 내용을 key=value(쿼리 파라미터 형식)으로 http 메세지 바디를 통해 서버에 전달
    + 전송 데이터를 url encoding 처리
- 회원 가입, 상품 주문, 데이터 변경
<br>

##### GET 방식의 경우
submit 버튼을 누르면 GET 방식은 메세지 바디를 사용하지 않기 때문에 오른쪽과 같이 http 메세지를 만들 때 쿼리 파라미터에 넣는 형식으로 생성하여 서버에 전달한다.
<img src="https://user-images.githubusercontent.com/108064146/208844495-4f1bfc96-1f96-4891-992f-f9e56152fe4a.png">

##### 파일과 같이 전송해야하는 경우
메세지 바디에 넣는 데이터 형식으로 `multipart/form-data` 타입을 사용하여 http 메세지를 생성한다.
<img src="https://user-images.githubusercontent.com/108064146/208845239-954f81b9-3120-483e-aefd-91b62d7da877.png">

- 주로 파일 업로드와 같은 바이너리 데이터를 전송할 때 사용한다. (다른 종류의 여러 파일과 폼의 내용도 함께 전송)
- `10923...` : 이미지에 대한 바이트 정보

> #### 주의: HTML Form을 통한 데이터 전송은 GET, POST만 지원한다.

#### 4. HTTP API를 통한 데이터 전송의 경우
클라이인트에서 서버로 데이터를 바로 전송해야하는 경우 <br>
HTML Form을 사용하지 않는 거의 모든 상황에서 사용한다.
<img src="https://user-images.githubusercontent.com/108064146/208850971-d6c47535-2ebc-41e8-b8c7-1cdf62283c77.png">

- 서버 to 서버
    + 백엔드 시스템 통신
- 앱 클라이언트
    + 아이폰 안드로이드
- 웹 클라이언트
    + HTML에서 Form 전송 대신 js를 통한 통신(AJAX 통신)
- GET, POST, PUT, PATCH, DELETE 모두 사용 가능
- Content-Tpye: `application/json`을 주로 사용 (사실상 표준)
- 회원 가입, 상품 주문, 데이터 변경

## HTTP API URI 설계 예시
상황 : 회원 관리 시스템을 만드는 상황

- 회원 목록 : /members -> GET
- 회원 등록 : /members -> POST
- 회원 조회 : /members/{id} -> GET
- 회원 수정 : /members/{id} -> PATCH, PUT, POST
- 회원 삭제 : /members/{id} -> DELETE

### 신규 자원 등록 POST vs PUT
대부분 POST 방식을 사용한다.

#### POST - 신규 자원 등록 특징
- 클라이언트는 등록될 리소스의 URI를 알지 못한다.
    + 클라이언트가 회원 등록을 위해 POST /members의 URI로 데이터를 전송하면 서버가 DB에 저장해서 새로 등록된 리소스 URI를 생성해준다.
    + 이러한 방식을 컬렉션이라고 한다.

``` html
HTTP/1.1 201 Created
Location: /members/100
```

#### collection
+ 서버가 관리하는 리소스 디렉토리
+ 서버가 리소스의 URI를 생성하고 관리
+ 여기서 컬렉션은 /members 이다.

#### PUT 기반 등록
- 파일 목록 : /files -> GET
- 파일 조회 : /files/{filename} -> GET
- 파일 등록 : /files/{filename} -> PUT
- 파일 삭제 : /files/{filename} -> DELETE
- 파일 대량 등록 : /files -> POST

#### PUT 기반 등록 특징
- 클라이언트가 리소스 URI를 알고 있어야 한다.
    + PUT /files/star.jpg
- 클라이언트가 직접 리소스의 URI를 지정한다.
- 이런 방식을 store라고 한다.

#### store
+ 클라이언트가 관리하는 리소스 저장소
+ 클라이언트가 리소스의 URI를 알고 관리
+ 여기서 스토어는 /files 이다.

## HTML FORM 사용
HTML FORM은 GET, POST 만 지원 (AJAX 같은 기술을 사용해서 해결할 수 도 있지만 순수 HTML FORM 만 사용한다고 가정)

- 회원 목록 : /members -> GET
- 회원 등록 폼 : /members/new -> GET
- 회원 등록 : /members/new, /members -> POST
- 회원 조회 : /members/{id} -> GET
- 회원 수정 폼 : /members/{id}/edit -> GET
- 회원 수정 : /members/{id}/edit, /members/{id} -> POST
- 회원 삭제 : /members/{id}/delete -> POST (DELETE를 사용할 수 없기 때문에 어쩔 수 없이 컨트롤 URI를 사용한다.)

### 컨트롤 URI 
HTML FORM의 예시의 경우 : /new, /edit, /delete 
+ `문서`, `컬렉션`, `스토어`로 해결하기 어려운 추가 프로세스 실행의 경우 사용한다.
+ HTTP API URI를 설계할 때도 사용.
+ 최대한 사용가능한 메서드를 사용하고 안될 경우 컨트롤 URI를 사용한다. (굉장히 빈번함)
+ 동사로 사용해야한다.

### 문서 (document)
- 단일 개념 (파일 하나, 객체 인스턴스, 데이터 베이스 row)
- /members/100, /files/star.jpg

## Reference
- 모든 개발자를 위한 HTTP 웹 기본 지식 [김영한]

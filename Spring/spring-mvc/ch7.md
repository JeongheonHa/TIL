# MVC 활용

## 1. static vs templates
- /resources/static 폴더에는 정적 리소스를 넣어준다. 해당 리소스의 경로를 url로 요청하면 해당 리소스를 웹 브라우저에 전달한다.
- /resources/templates 폴더에는 동적 리소스를 넣어준다. 해당 리소스는 컨트롤러를 통해 뷰 렌더링 과정을 통해 접근할 수 있다.
- static, templates 모두 리소스의 전체 url을 요청하면 웹 브라우저에 해당 리소스를 응답해준다. (뷰 렌더링 x)
- url로 접근하는 경우 서버를 띄우지 않아도 접근하여 즉각적인 반응을 확인할 수 있다. 

## 2. 추가 애노테이션
- @RequiredArgsConstructor: final이 붙은 멤버변수만 사용해서 생성자를 자동으로 만들어준다.
- @PostConstruct: 해당 빈의 의존관계가 모두 주입되고 나면 초기화 용도로 호출된다. (테스트 용)

## 3. thymeleaf

### 3.1 타임리프 사용 선언
```html
<html xmlns:th="http://www.thymeleaf.org">
```

### 3.2 속성 변경
- `th:xxx` : 타임리프 뷰 템플릿을 거치면 기존의 xxx 속성을 th:xxx 속성으로 덮어버린다. 만약 값이 없다면 새로 생성한다.
- 따라서, 뷰 렌더링을 거치지 않고 url로 접근하면 xxx 속성이 사용되고, 뷰 렌더링을 거치면 th:xxx 속성이 사용된다. (th: 속성을 알지 못하므로 무시)
- th:xxx는 서버 사이드에서 렌더링된다.

#### th:href
```html
<link th:href="@{/css/bootstrap.min.css}"
        href="../css/bootstrap.min.css" rel="stylesheet">
```

#### th:onclick
```html
<button class="w-100 btn btn-primary btn-lg"
        onclick="location.href='editForm.html'"
        th:onclick="|location.href='@{/basic/items/{itemId}/edit(itemId=${item.id})}'|"
        type="button">상품 수정</button>
```

#### th:value
value속성을 th:value로 변경

#### th:action
HTML Form 에서 th:action에 action경로를 생략하면 현재 URL에 데이터를 전송한다.
- 따라서, 하나의 URL로 등록 폼(GET)과 등록 처리(POST)를 처리할 수 있다.

### 3.3 URL 링크 표현식
- `@{...}`: url 링크를 표현할 때는 해당 형식을 사용해야한다.
```html
th:href="@{/css/bootstrap.min.css}"
```

### 3.4 리터럴 대체
- `|...|`: 타임 리프에서 문자와 표현식 등은 분리되어 있어 + 로 더해서 사용해야지만 리터럴 대체 문법을 사용하면 더하기 없이 사용할 수 있다.

### 3.5 반복 출력
- `th:each`: 모델에 있는 items에서 하나씩 꺼내서 루프를 돈다.
```html
<tr th:each="item : ${items}">
    <td><a href="item.html" th:href="@{/basic/items/{itemId}(itemId=${item.id})}" th:text="${item.id}">회원id</a></td>
    <td><a href="item.html" th:href="@{|/basic/items/${item.id}|}" th:text="${item.itemName}">상품명</a></td>
    <td th:text="${item.price}">10000</td>
    <td th:text="${item.quantity}">10</td>
</tr>
```

### 3.6 변수 표현식 
- `${...}`: 프로퍼티 접근법을 통해 모델에 포함된 값이나, 타임리프 변수로 선언한 값을 조회할 수 있다.
```html
th:text="${item.price}"
```
- 위에 처럼 사용하면 모델로 넘어온 item데이터의 price를 조회해 상황에 맞게 getPrice() 또는 setPrice() 형식으로 호출해준다.

### 3.7 내용 변경
- `th:text`: 내용의 값을 th:text="---" 값으로 변경한다.
```html
<td th:text="${item.price}">10000</td> // 10000 을 해당 변수로 치환한다.
```


### 3.8 URL 링크 표현식
- `{itemId}(itemId=${item.id}, query='test')`: 뷰 렌더링을 통해 값이 들어오면 해당 경로 변수에 값이 치환된다. `,` 뒤의 나머지는 url에 쿼리 파라미터로 들어간다.
```html
th:href="@{/basic/items/{itemId}(itemId=${item.id})}"

// 리터럴 대체 사용
th:href="@{|/basic/items/{itemId}|}"
```

## 4. @ModelAttribute의 두가지 기능
- 요청 파라미터의 값을 프로퍼티 접근법(setXxxx)으로 받아와 객체를 생성하고 해당 객체에 값을 넣어준다. (이전에 배운거)

- Model에 @ModelAttribute로 지정한 객체를 자동으로 넣어준다.
    + model.addAttribute("item", item)를 대신 처리해준다.
    + model에 지정할 이름은 @ModelAttribute("이름")으로 지정하거나 ("이름")을 생략하면 해당 객체 타입의 첫글자만 소문자로 바꿔서 이름으로 지정한다. 
    + 따라서 @ModelAttribute 자체를 생략하면 Model에 Item -> item의 이름으로 데이터 객체를 저장한다.

```java
    @PostMapping("/add")
    public String addItemV2 (@ModelAttribute("item") Item item) {
        itemRepository.save(item);
//      model.addAttribute("item", item); // 자동으로 추가해주기 때문에 생략 가능
        return "basic/item";
    }
```

## 5. 리다이렉트
`return "redirect:/경로";` : 해당 경로로 리다이렉트 한다.
- 리다이렉트 경로로 경로 변수({itemId})의 값을 그대로 사용 가능하다.
    + return "redirect:/basic/items/{itemId}";

```java
@GetMapping("/{itemId}/edit")
public String editForm(@PathVariable Long itemId, Model model) {
    Item item = itemRepository.findById((itemId));
    model.addAttribute("item", item);
    return "basic/editForm";
}

@PostMapping("/{itemId}/edit")
public String edit(@PathVariable Long itemId, @ModelAttribute Item item) {
    itemRepository.update(itemId, item);
    return "redirect:/basic/items/{itemId}";
}
```

### 5.1 PRG 방식 (Post/Redirect/Get)
상품 등록 폼에서 상품을 등록(POST)하고 새로고침을 하면 이전에 요청한 상품 등록(POST)를 다시 호출하여 같은 상품을 반복해서 등록하게 된다. 이것을 방지하기 위해 상품 등록을 하고 난 후 상품 상세 폼으로 갈 때 바로 뷰 렌더링하는 것이 아닌 redirect로 웹 브라우저에서 해당 상품 상세 폼을 GET방식으로 다시 요청해 상품 상세 폼을 받음으로써 해결할 수 있다.
```java
@PostMapping("/add")
public String addItemV6 (Item item, RedirectAttributes redirectAttributes) {
    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);
    return "redirect:/basic/items/{itemId}";
}

@GetMapping("/{itemId}")
public String item(@PathVariable Long itemId, Model model) {
    Item item = itemRepository.findById(itemId);
    model.addAttribute("item", item);
    return "basic/item";
}
```

### 5.2 RedirectAttributes
리다이렉트와 관련된 속성을 지정해준다.
RedirectAttributes를 사용하면 URL 인코딩, pathVariable, 쿼리 파라미터까지 처리해준다.
- redirectAttributes.addAttribute("itemId", savedItem.getId()); : "itemId"의 값이 redirect 경로의 상대 변수({itemId})에 치환된다.
- redirectAttributes.addAttribute("status", true); : redirect 경로에 치환될 수 없는 것들은 쿼리 파라미터로 넘어간다.

#### 상품 등록을 한 경우 상품 등록이 잘 저장되었다고 표시하는 법
- `th:if`: 해당 조건이 참이면 실행한다.
- 타임 리프는 쿼리 파라미터를 조회할 수 있는 예약어 `param`을 지원한다.
- 현재 url의 쿼리 파라미터(?status=true)의 값을 가져와 true인 경우에만 해당 텍스트가 나타나게 구현한다.

```html
// 상품 상세 폼
<h2 th:if="${param.status}" th:text="'저장 완료'"></h2>
```

## Reference
> 스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술 [김영한]

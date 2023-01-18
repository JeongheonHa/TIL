# 타임 리프 스프링 통합 기능

## 1. 입력 폼 처리
- `th:object` - 커맨드 객체 생성
- `*{...}` - 선택 변수 식 (${item.itemName}과 같다.)
- `th:field` - id, name, value속성을 자동 처리

```html
<form action="item.html" th:action th:object="${item}" method="post">
        <div>
            <label for="itemName">상품명</label>
            <input type="text"  th:field="*{itemName}"class="form-control" placeholder="이름을 입력하세요">
        </div>
```
- 폼에 넘어온 데이터 중 커맨드 객체를 생성
- 선택 변수 식을 이용해 커맨드 객체의 필드에 접근
- th:field 속성으로 해당 필드의 id(=변수 이름), name(=변수 이름), value(=변수 값)를 자동 생성

## 2. 체크 박스
- 체크 박스 체크시 HTML Form에서 open=on이라는 값이 넘어가고 스프링의 타입 컨버터가 on을 true로 변환
- HTML에서 체크 박스를 선택하지 않고 폼을 전송하면 open 필드 자체가 넘어가지 않는다.
- `th:field`가 폼에 _open=on이라는 히든 필드를 생성해 보낸다.
- 스프링 MVC가 히든 필드를 인식해서 open이 true/false로 인식할 수 있게 한다.
- `open=on&_open=on`: open=true 로 인식
- `_open=on`: open=false 로 인식
- `checked="checked"`: th:field를 사용하면, 값이 true인 경우 체크를 자동을 처리해준다.
```html
<div>
    <div class="form-check">
        <input type="checkbox" id="open" th:field="*{open}" class="form-check-input">
        <!-- <input type="hidden" name="_open" value="on"/> --> 
        <label for="open" class="form-check-label">판매 오픈</label>
    </div>
</div>
```

### 2.1 다중 체크 박스
- ``${#ids.prev('regions')}`: th:field로 생성된 id를 자동으로 생성해준다. (ex. id="regions1", id="regions2", id="regions3")
- th:each의 checkedbox의 경우 th:field는 자신이 생성한 value와 th:value를 비교해서 체크를 처리하기 때문에 th:value를 지정해줘야한다.
```html
<div>등록 지역</div>
    <div th:each="region : ${regions}" class="form-check form-check-inline">
        <input type="checkbox" th:field="*{regions}" th:value="${region.key}" class="form-check-input">
        <label th:for="${#ids.prev('regions')}"
                th:text="${region.value}" class="form-check-label">서울</label>
    </div>
```

#### @ModelAttribute
- @ModelAttrbute 애노테이션을 컨트롤러의 별도의 메서드에 적용하면 해당 컨트롤러를 요청할 때 마다 "regions"이름의 반환 객체(regions)를 model에 자동 저장한다.
```java
@ModelAttribute("regions")
public Map<String, String> regions() {
    Map<String, String> regions = new LinkedHashMap<>();
    regions.put("SEOUL", "서울");
    regions.put("BUSAN", "부산");
    regions.put("JEONJU", "전주");
    return regions;
}
```

## 3. 라디오 버튼
라디오 버튼은 체크 박스와 달리 이미 선택되어 있으면 수정시에도 항상 하나를 선택하도록 되어 있으므로 히든 필드가 필요하지 않다.

```html
<!-- radio button (Enum 사용) -->
<div>
    <div>상품 종류</div>
    <div th:each="type : ${itemTypes}" class="form-check form-check-inline">
        <input type="radio" th:field="*{itemType}" th:value="${type.name()}" class="form-check-input">
        <label th:for="${#ids.prev('itemType')}" th:text="${type.description}" class="form-check-label">BOOK</label>
    </div>
</div>
```

## 4. select box
선택시 selected="selected" 유지된다.
```html
<!-- SELECT (객체 사용)-->
<div>
    <div>배송 방식</div>
    <select th:field="*{deliveryCode}" class="form-select">
        <option value="">==배송 방식 선택==</option>
        <option th:each="deliveryCode : ${deliveryCodes}" th:value="${deliveryCode.code}"
                th:text="${deliveryCode.displayName}">FAST</option>
    </select>
</div>
```

## Reference
> 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술 [김영한]

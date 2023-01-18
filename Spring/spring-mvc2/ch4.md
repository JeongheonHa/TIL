# 검증 (Validation)
상품 등록시 조건에 맞지 않는 입력이 입력되면 오류 페이지가 아닌 고객에게 다시 상품 등록 폼을 보여주고 무엇이 잘못되었는지 알려줘야한다.

## 1. 검증 직접 처리하기 - v1

```java
@PostMapping("/add")
public String addItem(@ModelAttribute Item item, RedirectAttributes redirectAttributes, Model model) {

    //검증 오류 결과를 보관
    Map<String, String> errors = new HashMap<>();

    //검증 로직
    // 필드 오류
    if (!StringUtils.hasText(item.getItemName())) {
    errors.put("itemName", "상품 이름은 필수입니다.");
    }

    // 글로벌 오류
    if (item.getPrice() != null && item.getQuantity() != null) {
        int resultPrice = item.getPrice() * item.getQuantity();
        if (resultPrice < 10000) {
            errors.put("globalError", "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice);
        }
    }

    //검증에 실패하면 다시 입력 폼으로 
    if (!errors.isEmpty()) {
        model.addAttribute("errors", errors);
        return "validation/v1/addForm";
    }

    // 성공 로직
    ...

}
```

### 1.1 입력 폼에 검증 추가
```html
<style>
    .container {
        max-width: 560px;
    }
    .field-error {  // 검증시 스타일 추가
        border-color: #dc3545;
        color: #dc3545;
    }
</style>
```
```html
<form action="item.html" th:action th:object="${item}" method="post">

    <!-- 글로벌 오류 -->
    <div class="field-error" th:if="${errors?.containsKey('globalError')}">
        <p class="field-error" th:text="${errors['globalError']}">전체 오류 메세지</p>
    </div>

    <!-- 필드 오류 -->
    <div>
        <label for="itemName" th:text="#{label.item.itemName}">상품명</label>
        <input type="text" id="itemName" th:field="*{itemName}"
                th:class="${errors?.containsKey('itemName')} ? 'form-control field-error' : 'form-control'"
                class="form-control" placeholder="이름을 입력하세요">
        <div class="field-error" th:if="${errors?.containsKey('itemName')}" th:text="${errors['itemName']}">
            상품명 오류
        </div>
    </div>
```
- Safe Navigation Operator(`?.`): errors가 null일 때 NullPointerException 대신 null을 반환한다.
- th:if에서 null은 실패 처리가 되서 오류 메세지가 출력되지 않는다.

### 1.2 직접 검증 처리의 문제점
- 뷰 템플릿에서 중복 처리가 많다.
- 타입 오류 발생시 오류 페이지로 간다.
- 이전에 입력했던 내용을 유지해야한다.

## 2. BindingResult 사용 - v2
- 해당 item 객체에 바인딩된 결과가 BindingResult에 담긴다. (이전 버전의 errors 역할)
- @ModelAttribute 바로 다음에 와야한다.
- BindingResult는 자동으로 뷰로 넘어가기 때문에 model에 담지 않아도 된다.
- BindingResult는 인터페이스고 구현체인 BeanPropertyBindingResult가 사용된다.

```java
@PostMapping("/add")
public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

    // 검증 로직
    // 필드 오류
    if (!StringUtils.hasText(item.getItemName())) {
        bindingResult.addError(new FieldError("item", "itemName", "상품 이름은 필수입니다."));
    }


    // 글로벌 오류
    if (item.getPrice() != null && item.getQuantity() != null) {
        int resultPrice = item.getPrice() * item.getQuantity();
        if (resultPrice < 10000) {
            bindingResult.addError(new ObjectError("item", "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));
        }
    }

    // 검증에 실패하면 다시 입력 폼으로 되돌아가기
    if (bindingResult.hasErrors()) {
        log.info("errors = {}", bindingResult);
        return "validation/v2/addForm";
    }

    // 성공 로직
    ...
}
```

### 2.1 FieldError, ObjectError
- objectName: @ModelAttribute 이름
- field: 오류가 발생한 필드 명
- defaultMessage: 기본 메세지
- rejectedValue: 사용자가 입력한 값(거절된 값)
- bindingFailure: 타입 오류 같은 바인딩 실패인지, 검증 실패인지 구분
```java
// 필드 오류
public FieldError(String objectName, String field, String defaultMessage) {}
public FieldError(String objectName, String field, @Nullable Object rejectedValue, boolean bindingFailure, 
                @Nullable String[] codes, @Nullable Object[] arguments, @Nullable String defaultMessage)
// 글로벌 오류
public ObjectError(String objectName, String defaultMessage) {}
public ObjectError(String objectName, @Nullable String[] codes, @Nullable Object[] arguments, @Nullable String defaultMessage)
```

### 2.2 입력 폼에 검증 추가
- `${#fields.---}`: BindingResult에 접근
- `${#fields.hasGlobalErrors()}`: BindingResult에 글로벌 오류가 있는지 확인
- `${#fields.globalErrors()}`: BindingResult에서 글로벌 오류를 가져온다.
- `th:errorclass="field-error"`: th:field에서 지정한 필드에 오류가 있으면 class 정보 추가
- `th:errors="*{itemName}"`: 해당 필드에 오류가 있는 경우 태그를 출력 (th:if 편의 버전)

```html
<form action="item.html" th:action th:object="${item}" method="post">

    <div th:if="${#fields.hasGlobalErrors()}">
        <p class="field-error" th:each="err : ${#fields.globalErrors()}" th:text="${err}">전체 오류 메세지</p>
    </div>

    <div>
        <label for="itemName" th:text="#{label.item.itemName}">상품명</label>
        <input type="text" id="itemName" th:field="*{itemName}"
                th:errorclass="field-error" class="form-control" placeholder="이름을 입력하세요">
        <div class="field-error" th:errors="*{itemName}">
            상품명 오류
        </div>
    </div>
```

### 2.3 검증의 2가지 상황
검증은 크게 2가지로 나눌 수 있다.
- 클라이언트가 잘못된 타입을 입력하는 경우
- 클라이언트가 비즈니스 로직에 위배된 입력을 한 경우

#### 클라이언트가 잘못된 타입을 입력하는 경우 (타입 오류)
BindingResult가 있으면 @ModelAttribute에 데이터 바인딩시 오류가 발생해도 스프링이 해당 오류 정보를 갖는 FieldError를 생성해 BindingReulst객체에 담은 후 컨트롤러가 호출된다.
- 타입 오류가 발생하면 오류 페이지로 넘어가는 부분 해결

### 2.4 v2의 문제점
- 여전히 이전 입력 값이 사라진다.

## 3. FieldError와 ObjectError의 추가 파라미터 사용

### 3.1 거절된 값 보관하기 - v3
이전에 입력한 값이 사라지는 부분을 해결하기 위해선 거절된 값을 따로 보관해야한다. 만약 타입 오류가 발생하면 Intger 타입에 String 타입을 담을 수 없기 때문에 이전에 입력한 값을 유지할 수 없다.
```java
@PostMapping("/add")
public String addItemV2(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
    if (!StringUtils.hasText(item.getItemName())) {
        bindingResult.addError(new FieldError("item", "itemName", item.getItemName(), false, null, null, "상품 이름은 필수입니다.")); 
    }
}
```
- FieldError는 오류 발생시 사용자 입력 값을 저장하는 기능을 제공 (rejectedValue)

### 3.2 타임리프에서 입력 값 유지
`th:field`는 정상 상황에는 모델 객체의 값을 사용하지만, 오류가 발생하면 FieldError에서 보관한 rejectedValue를 사용해서 출력한다.

### 3.3 오류 코드와 오류 메세지 관리
FieldError와 ObjectError의 codes와 argments 파라미터를 이용해 오류 메세지를 한 곳에서 관리할 수 있다.
- `spring.messages.basename=messages,errors`: messages와 errors를 기본 메세지 소스로 등록한다.
- codes는 순서대로 매칭해서 처음 매칭되는 메세지를 사용한다. 해당 배열에서 못찾으면 defaultMessage가 출력된다.

```html
range.item.price=가격은 {0} ~ {1} 까지 허용합니다. 
```
```java
if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
    bindingResult.addError(new FieldError("item", "price", item.getPrice(), false, new String[]{"range.item.price"}, new Object[]{1000, 1000000}, null));
}
```

### 3.4 rejectValue(), reject()
사실 BindingResult 객체는 검증 해야할 객체 바로 다음에 오기 때문에 검증 객체를 알고 있다. <br>
따라서 FieldError, ObjectError를 직접 생성하지 않고 rejectValue(), reject() 메서드를 사용하면 간편하게 검증 오류를 다룰 수 있다.
- 사실, rejectValue(), reject() 내부에서 FieldError, ObjectError 객체를 대신 생성해준다.
- `errorCode`: 오류 코드 (MessageCodesResolver가 object 명 + field 명을 조합해서 적용해준다.)
- `errorArgs`: {} 치환 값

```java
void rejectValue(@Nullable String field, String errorCode, @Nullable Object[] errorArgs, @Nullable String defaultMessage);
void reject(String errorCode, @Nullable Object[] errorArgs, @Nullable String defaultMessage);
```
```java
// 필드 오류
bindingResult.rejectValue("price", "range", new Object[]{1000, 1000000}, null)

// 글로벌 오류
bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
```
### 3.5 MessageCodesResolver
검증 오류 코드로 메세지 코드들을 생성한다.
- MessageCodesResolver는 인터페이스이고 DefaultMessageCodesResolver는 기본 구현체이다.

#### 기본 멧세지 생성 규칙
```html
객체 오류
1. code + "." + object name // 오브젝트
2. code

필드 오류
1. code + "." + object name + "." + field   // 오브젝트 + 필드
2. code + "." + field       // 필드
3. code + "." + field type  // 필드 타입
4. code
```

#### 동작 방식
1. rejectValue(), reject() 내부에서 MessageCodesResolver를 사용해서 메세지 코드를 생성
2. FieldError, ObjectError 객체를 생성해 생성된 메세지 코드들을 순서대로 넣고 보관
3. th:errors가 true면 errors.properties에 생성된 codes를 순차적으로 돌면서 가장 먼저 발견된 code의 오류 메세지를 출력

### 3.6 ValidationUtils
Empty 또는 Whitespace 오류 검증 로직
```java
// 사용 전
if (!StringUtils.hasText(item.getItemName())) { 
    bindingResult.rejectValue("itemName", "required", "기본: 상품 이름은 필수입니다."); 
}
// 사용 후
ValidationUtils.rejectIfEmptyOrWhitespace(bindingResult, "itemName", "required");
```

### 3.7 검증 로직 밖의 오류의 경우 (타입 오류)
MessageCodesResolver가 typeMismatch 오류 코드를 이용해서 생성
```html
typeMismatch.item.price 
typeMismatch.price 
typeMismatch.java.lang.Integer 
typeMismatch
```

- 에러 메세지 소스에 추가
```html
typeMismatch.java.lang.Integer=숫자를 입력해주세요. 
typeMismatch=타입 오류입니다.
```

### 3.8 v3의 문제점
- 하나의 컨트롤러에 검증 로직과 비즈니스 로직이 섞여있다.

## 4. Validator - v4
해당 검증 로직을 갖는 검증기를 따로 만들어 컨트롤러로부터 분리할 수 있다.
```java
public interface Validator {
    boolean supports(Class<?> clazz);
    void validate(Object target, Errors errors);
}
```
- `supports()`: 해당 검증기를 지원하는 여부 확인
- `validate(Object target, Errors errors)` : 검증 대상 객체와 BindingResult

### 4.1 검증기 만들어 직접 호출하기

```java
@Component  // 검증기 빈으로 등록해서 보관
public class ItemValidator implements Validator {

    @Override
    public boolean supports(Class<?> clazz) {
        return Item.class.isAssignableFrom(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {

        Item item = (Item) target;

        // 검증 로직
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "itemName", "required");
    }
}
```
```java
// 컨트롤러

// 검증기 주입
private final ItemValidator itemValidator;
// 검증기 직접 호출
itemValidator.validate(item, bindingResult);
```

### 4.2 검증기 자동 호출
Validation 인터페이스를 사용해서 검증기를 만들면 검증기를 자동 호출할 수 있다.
- `@InitBinder`: @Controller나 @ControllerAdvice가 붙은 클래스는 @InitBinder가 붙은 메서드를 가질 수 있으며 이는 WebDataBinder 인스턴스를 초기화하는 메세드이다. 
- `WebDataBinder`: 요청 파라미터를 자바 빈 객체로 바인딩하는 역할을 하며 검증 기능도 내부에 포함한다.

```java
@InitBinder
public void init(WebDataBinder dataBinder) {
    dataBinder.addValidators(itemValidator);
}
```
- @InitBinder를 사용해 WebDataBinder에 적용하고 싶은 검증기를 넣어준다.
- WebDataBinder에 검증기를 추가하면 해당 컨트롤러에서 검증기를 자동으로 적용한다.(해당 컨트롤러만 영향을 준다.)
- WebDataBinder 객체는 컨트롤러 호출시 마다 새로 생성된다.

#### @Validated
검증기를 실행하라는 애노테이션이다.
- WebDataBinder에 등록한 검증기들에서 해당하는 검증기를 찾아서 실행한다.
```java
@PostMapping("/add")
public String addItemV6(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

}
```

#### 동작 방식
1. `supports()`를 이용해 해당 객체(Item)에 맞는 검증기를 찾는다.
2. Item 검증기를 찾았으면 `validate()`를 이용해 검증기 실행

#### 글로벌 설정
검증기를 해당 컨트롤러에만 적용시키는 것이 아닌 글로벌로 모든 컨트롤러에 설정할 수 있지만 잘 사용하지 않는다. 
- 스프링 부트가 자동으로 더 좋은 BeanValidator를 글로벌 Validator로 등록해준다.

## Reference
> 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술 [김영한]

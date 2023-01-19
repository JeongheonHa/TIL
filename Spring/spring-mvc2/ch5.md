# Bean Validation
일반적인 검증 로직을 공통화하고 표준화한 것
- Bean Validation은 특정 구현체가 아닌 Bean Validation 2.0(JSR-380) 기술 표준이다.
- 구현체는 Hibernate Validator이다. (ORM과는 관련이 없다.)

## 1. 의존 관계 세팅
스프링 부트가 해당 라이브러리를 넣으면 자동으로 Bean Validator를 인지하고 스프링에 통합한다.
```html
implementation 'org.springframework.boot:spring-boot-starter-validation'
```
- 스프링 부트는 자동으로 LocalValidatorFactoryBean을 글로벌 Validator로 등록해주며 @Validated를 적용해 검증기를 사용하기만 하면 된다.
- `LocalValidatorFactoryBean`: 애노테이션(@NotNull같은) 기반으로 모든 걸 검증할 수 있는 Validator

## 2. 검증 애노테이션
- `@NotBlank`: 빈값 + 공백만 있는 경우를 허용하지 않는다.
- `@NotNull`: null 을 허용하지 않는다.
- `@Range(min = 1000, max = 1000000)`: 범위 안의 값이어야 한다. 
- `@Max(9999)`: 최대 9999까지만 허용한다.

## 3. Bean Validation에서 타입 오류
BeanValidator는 바인딩에 실패한 필드는 BeanValidation을 적용하지 않는다.
- ex. price에 문자 입력 -> 타입 변환 실패 -> typeMismatch FieldError 추가 -> price필드는 BeanValidation 적용 X

## 4. 에러 코드
오류 코드가 애노테이션 이름으로 등록된다.
```html
@NotBlank
NotBlank.item.itemName 
NotBlank.itemName 
NotBlank.java.lang.String 
NotBlank
```
- `{0}`은 필드 명이다.
```html
<!--- 메세지 소스 등록 --->
NotBlank={0} 공백X 
Range={0}, {2} ~ {1} 허용 
Max={0}, 최대 {1}
```

### 4.1 메세지 찾는 순서
1. 생성된 메세지 코드 순서대로 메세지 소스에서 메세지를 찾는다.
2. 애노테이션의 message 속성의 메세지를 사용한다. (ex. @NotBlank(message = "공백! {0}"))
3. 라이브러리가 제공하는 기본 메세지를 사용한다.

## 5. 오브젝트 오류
해당 객체에 `@ScriptAssert`를 사용하면 되지만 검증 기능이 해당 객체의 범위를 넘어서는 경우가 빈번히 발생하는데 그런 경우 대응이 어렵다. <br>
따라서, 오브젝트 오류는 오브젝트 오류 관련 부분만 직접 자바 코드로 작성하는 것이 더 좋다.

```java
// 비추
@Data
@ScriptAssert(lang = "javascript", script = "_this.price * _this.quantity >= 10000")
public class Item {
//...
}

// 추천
bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
```

## 6. Bean Validation의 한계
대부분 상품을 등록할 때와 수정할 때의 요구조건이 다르다. <br>
하지만 Bean Validation은 해당 객체의 필드에 직접 애노테이션을 적용시키기 때문에 요구 조건을 분리할 수 없다.

### 6.1 요구 조건을 분리하는 2가지 방법
1. BeanValidation의 `groups` 기능 (사용 안함)
2. 해당 객체를 직접 사용하는 것이 아닌 폼 전송을 위한 별도의 모델 객체를 만들어서 사용 (강추)
- 보통 요구 조건 뿐만 아니라 데이터 또한 등록할 때와 수정할 때 데이터가 완전 다르기 때문에 용도에 맞는 폼 전송 객체를 별도로 만들어 사용해야한다.

### 6.2 groups

#### 그룹 인터페이스를 만든다.
```java
public interface SaveCheck {}

public interface UpdateCheck {}
```

#### 해당 객체의 필드에 그룹을 지정한다.
```java
@NotNull(groups = {SaveCheck.class, UpdateCheck.class})
```

#### 해당 로직에 그룹을 적용한다.
```java
@PostMapping("/add")

public String addItemV2(@Validated(SaveCheck.class) @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
}
```

### 6.3 Form 전송 객체 분리

#### Form 데이터 전달 과정
- HTML Form -> ItemSaveForm -> Controller -> 해당 데이터를 갖는 Item 생성 -> Repository

#### Form 전송 객체 분리
```java
@Data
public class ItemSaveForm {
    @NotBlank
    private String itemName;
    @NotNull
    @Range(min = 1000, max = 1000000)
    private Integer price;
    @NotNull
    @Max(value = 9999)
    private Integer quantity;
}
```

#### Form에서 가져오는 객체를 Form 전송 객체로 바꾼다.
```java
@PostMapping("/add")
public String addItem(@Validated @ModelAttribute("item") ItemSaveForm form, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
    Item item = new Item(form.getItemName(), form.getPrice(), form.getQuantity()); 
    Item savedItem = itemRepository.save(item);
}
```

## 7. HTTP API 검증

### 7.1 HttpMessageConverter
@Validated는 HttpMessageConverter에도 적용할 수 있다.

### 7.2 API의 2가지 검증 상황
- 실패 요청: JSON을 객체로 생성하는 것 자체가 실패 (타입 오류)
- 검증 오류 요청: JSON을 객체로 생성하는 것은 성공했고, 검증에서 실패 (검증 로직 오류)

#### 타입 오류
HttpMessageConverter에서 요청 JSON을 ItemSaveForm 객체로 생성하는데 실패한다. <br>
이 경우 컨트롤러 자체가 호출되지 않으며 그 전에 예외가 발생한다. 또한 Validator도 실행되지 않는다.

#### 검증 오류 요청
HttpMessageConverter에 의해 JSON 형태의 요청 데이터로 Form 객체를 생성하고 JSON 형식으로 오류 정보를 제공
- 실제로 할 때는 필요한 오류 정보만 뽑아서 출력한다.

```java
@ResponseBody
@PostMapping("/add")
public Object addItem(@RequestBody @Validated ItemSaveForm form, BindingResult bindingResult) {

    // 검증 로직
    if (bindingResult.hasErrors()) {
        return bindingResult.getAllErrors();
    }
    // 성공 로직 실행
    return form;
}
```

### 7.3 @ModelAttribute vs @RequestBody
- @ModelAttribute: 요청 파라미터를 처리할 때 각각의 필드 단위로 적용되기 때문에 특정 필드의 타입 오류가 발생해도 나머지 필드는 정상처리되어 검증기를 적용할 수 있다.
- @RequestBody: 요청 데이터를 처리할 때 HttpMessageConverter는 전체 단위로 적용되기 때문에 특정 필드의 타입 오류가 발생하면 이 후의 단계 자체가 진행되지 않고 예외가 발생한다. 컨트롤러도 호출되지 않고, 검증기도 적용할 수 없다.

## Reference
> 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술 [김영한]

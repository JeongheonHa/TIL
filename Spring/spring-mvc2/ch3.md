# 메세지, 국제화
- 메세지 기능: 다양한 메세지를 한 곳에서 관리하도록 하는 기능
- 국제화: 나라 별로 메세지 기능을 따로 관리하는 기능

## 1. 스프링의 메세지 소스 설정
메세지 관리 기능을 사용하기 위해선 스프링이 제공하는 `MessageSource`인터페이스에 `ResourceBundleMessageSource`구현체를 스프링 빈으르 등록하면 된다.
- 스프링 부트를 사용하면 자동으로 스프링 빈으로 등록한다.
```java
@Bean
public MessageSource messageSource() {
    ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
    messageSource.setBasenames("messages", "errors"); // 한번에 여러 파일 지정 가능
    messageSource.setDefaultEncoding("utf-8"); // encoding 정보 지정
    return messageSource;
}
```
### 1.1 설정 위치
- /resources/messages.properties가 기본 값으로 지정
- /resources/messages_en.properties로 추가로 국제화 기능을 적용

## 2. 메세지 소스 사용

```html
hello=안녕 
hello.name=안녕 {0}
```

```java
public interface MessageSource {
    // 메세지 소스를 읽어오는 기능 제공
    String getMessage(String code, @Nullable Object[] args, @Nullable String defaultMessage, Locale locale);
    String getMessage(String code, @Nullable Object[] args, Locale locale) throws NoSuchMessageException;   // 메세지가 없는 경우 NoSuchMessageException 예외 발생
}
```
- code: hello
- args: {0}
- locale: 언어 지정, locale정보를 기반으로 국제화 파일을 찾고 없으면 디폴트를 찾는다. (ex. Locale.KOREA)

## 3. 타임리프에서 사용
- `#{code}`(메세지 표현식): 스프링 메세지를 조회
```html
<h2 th:text="#{page.addItem}">상품 등록</h2>
```

### 3.1 파라미터가 있는 메세지 소스 사용
```html
<p th:text="#{hello.name(${item.itemName})}"></p>
```

## 4. 웹에서 실행
- 스프링은 언어 선택시 기본으로 `Accept-Language` 헤더의 값을 사용한다.

### 4.1 언어 선택 방식 변경
스프링은 Locale 선택 방식을 변경할 수 있도록 `LocaleResolver`라는 인터페이스를 제공한다.
```java
public interface LocaleResolver {
    Locale resolveLocale(HttpServletRequest request);
    void setLocale(HttpServletRequest request, @Nullable HttpServletResponse response, @Nullable Locale locale);
}
```
- 스프링은 기본으로 Accept-Language 방식의 `AcceptHeaderLocaleResolver` 구현체를 사용한다.
- 고객이 직접 Locale을 선택하도록 `쿠키나 세션 기반`의 Locale 선택 기능을 사용할 수도 있다.

## Reference
> 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술 [김영한]

# annotation

## 1. 애너테이션
- 주석처럼 프로그래밍 언어에 영향을 미치지 않으며, 유용한 정보를 제공하는 것
- 배경 : 옛날에는 소스코드를 작성하면 이 프로그램에 대한 문서를 따로 작성하였지만 사람들이 소스코드를 변경하면 문서도 내용을 변경해야하지만 소스코드만 바꾸고 문서 변경을 잘 하지않아   
버전 불일치가 발생하여 관리를 쉽게하기위해 문서와 소스코드를 하나로 합쳤으며 자바의 소스코드에 주석(/** ~ */)을 이용해서 문서의 내용을 넣고 javadoc.exe라는 프로그램이 주석을 추출해내서 문서를 따로 만들어냈다. 설정파일 또한 XML파일로 따로 만들어서 저장했었는데 소스코드안에 @Test와 같이 설정에 대한 정보를 넣었다. 이것이 바로 annotation이다.
@Test는 JUnit이라는 특정 프로그램을 위한 것이며 이 annotation이 의미를 갖는 특정 프로그램에서만 유효하다.

## 2. 애너테이션의 사용 예
```java
@Test   // 이 메서드가 테스트 대상임을 테스트 프로그램에게 알린다. (JUnit 단위 테스트 프로그램)
public void method() { ... }
```

## 3. 표준 애너테이션
- Java에서 제공하는 애너테이션

| 애너테이션 | 설명 |
|---|---|
| @Override | 컴파일러에게 오버라이딩하는 메서드라는 것을 알린다. |
| @Deprecated | 앞으로 사용하지 않을 것을 권장하는 대상에 붙인다. |
| @SuppressWarnings | 컴파일러의 특정 경고메시지가 나타나지 않게 해준다. (경고를 내가 확인했다는 의미도 가능) |
| @SafeVarargs | 지네릭스 타입의 가변인자에 사용한다. (jdk1.7) |
| @FunctionalInterface | 함수형 인터페이스라는 것을 알린다 (jdk1.8) |
| @Native | native메서드에서 참조되는 상수 앞에 붙인다 (jdk1.8) |

### 3.1 @Override
- 컴파일러가 같은 이름의 메서드가 조상에 있는지 확인하고 없으면 에러메시지 출력
```java
class Parent {
    void parentMethod() {}
}
class Child extends Parent {
    @Override
    void parentmethod() {}
}
```

### 3.2 @SuppressWarnings
- 경우에 따라서 경고가 발생할 것을 알면서도 묵인해야 할 때가 있는데, 이 경고를 그대로 놔두면 컴파일할 때마다 메시지가 나타난다.
- 자주 사용되는 경고 : `"deprecation"`, `"unchecked"`, `"rawtypes"`, `"varargs"` 정도
    - `"deprecation"` : @Deprecated가 붙은 대상을 사용했을 때 발생하는 경고
    - `"unchecked"` : 지네릭스로 타입을 지정하지 않았을 때 발생하는 경고
    - `"rawtypes"` : 지네릭스를 사용하지 않아서 발생하는 경고
    - `"varargs"` : 가변인자의 타입이 지네릭 타입일 때 발생하는 경고
```java
@SuppressWarnings("unchecked")
ArrayList list = new ArrayList();
list.add(obj);
```

- 둘 이상의 경고를 동시에 억제하는 경우
```java
@SuppressWarnings({"deprecation", "unchecked", "varargs"})
```

- 경고메시지의 종류를 알아보는 방법
: 터미널에서 `-Xlint`를 이용하면 `대괄호[]`안에 있는 것이 메시지의 종류이다.
```java
javac -Xlint 소스파일.java
```

### 3.3 SafeVarargs
- 메서드에 선언된 가변인자의 타입이 `non-reifiable`타입일 경우, 해당 메서드를 선언하는 부분과 호출하는 부분에서 발생하는 "unchecked" 경고를 억제하기 위해 사용
- 오버라이딩이 되지 않는 `static`, `final`이 붙은 메서드와 생성자에만 붙일 수 있다.
- `reifiable` : 컴파일 후에도 제거되지 않는 타입
- `non-reifiable` : 컴파일 후에 제거되는 타입 (대부분의 지네릭스 타입)
- 가능하면 항상 `@SafeVarargs`와 `@SuppressWarnings("Varargs")`를 같이 붙인다.

## 4. 메타 애너테이션
- 애너테이션을 위한 애너테이션
- 애너테이션의 정의 할 때 애너테이션의 `적용대상`이나 `유지기간` 등을 지정하는데 사용
- `java.lang.annotation`패키지에 포함

| 애너테이션 | 설명 |
|---|---|
| @Target | 애너테이션이 적용가능한 대상을 지정하는데 사용한다. |
| @Docummented | 애너테이션 정보가 javadoc으로 작성된 문서에 포함되게 한다. |
| @Inherited | 애너테이션이 자손 클래스에 상속되도록 한다. |
| @Retention | 애너테이션이 유지되는 범위를 지정하는데 사용한다. |
| @Repeatable | 애너테이션을 반복해서 적용할 수 있게 한다. (jdk1.8) |

### 4.1 @Target
- 애너테이션이 적용가능한 `적용 대상`을 지정
```java
// 애너테이션 만들어보기
import stataic java.lang.annotation.ElementType.*;  // ElementTpye.TYPE -> TYPE

@Target({FIELD, TYPE, TYPE_USE})    // 적용대상 지정
public @interface MyAnnotation { }  // MyAnnotation 정의

@MyAnnotation
class MyClass {
    @MyAnnotation
    int i;
    @MyAnnotation
    MyClass mc;
}
```
- 애너테이션 적용대상의 종류
: `java.lang.annotation.ElementType`이라는 열거형이 정의되어있다.

| 대상 타입 | 의미 |
|---|---|
| ANNOTATION_TYPE | 애너테이션 |
| CONSTRUCTOR | 생성자 |
| FIELD | 필드(멤버변수, enum 상수) |
| LOCAL_VARIABLE | 지역변수 |
| METHOD | 메서드 |
| PACKAGE | 패키지 |
| PARAMETER | 매개변수 |
| TYPE | 타입(클래스, 인터페이스, enum) |
| TYPE_PARAMETER | 타입 매개변수(jdk1.8) |
| TYPE_USE | 타입이 사용되는 모든곳(jdk1.8) |
> `주의` : `FIELD`는 `기본형`에 `TYPE_USE`는 `참조형`에 사용된다는 점을 주의하자

### 4.2 @Retention
- 애너테이션이 유지되는 기간의 지정하는데 사용

| 유지 정책 | 의미 |
|---|---|
| SOURCE | 소스 파일에만 존재, 클래스 파일에는 존재하지 않음 |
| CLASS | 클래스 파일에 존재, 실행시에 사용불가 (기본값) |
| RUNTIME | 클래스 파일에 존재, 실행시에 사용가능 |

- 컴파일러에 의해 사용되는 애너테이션의 유지 정책은 SOURCE이다.
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override { }  // 컴파일러를 위한 애너테이션의 유지기간은 소스파일까지이다.
```

- 실행시에 사용 가능한 애너테이션의 정책은 RUNTIME이다
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface FunctionalInterface { }
```

### 4.3 @Documented & @Inherited
- javadoc으로 작성한 문서(*.html)에 포함시키려면 `@Documented`를 붙인다.
- 애너테이션을 자손 클래스에 상속하고자 할 때, `@Inherited`를 붙인다.

### 4.4 @Repeatable
- 반복해서 붙일 수 있는 애너테이션을 정의할 때 사용
```java
@Repeatable(ToDos.class)
@interface ToDo {
    String value();
}
```

- @Repeatable이 붙은 애너테이션은 반복해서 붙일 수 있다.
```java
@ToDo("delete test codes.")
@ToDo("override inherited methods")
class MyClass {
    ...
}
```

- @Repeatable인 @ToDo를 하나로 묶을 컨테이너 애너테이션도 정의해야 함
```java
@interface ToDos {  // 여러 개의 ToDo애너테이션을 담을 컨테이너 애너테이션 ToDos
    ToDo[] value(); // ToDo애너테이션 배열타입의 요소를 선언, 이름이 반드시 value이어야 한다.
}
```

## 5. 애너테이션 타입 정의하기
- 애너테이션을 직접 만들어 사용할 수 있다. (@제외하고 interface 정의와 같다.)
```java
@interface 애너테이션이름 {
    // 애너테이션의 요소 선언
    타입 요소이름();    // 추상메서드
    ...
}
```

- 애너테이션의 메서드는 추상 메서드이며, 애너테이션을 적용할 때 지정(요소를 지정해주는 순서 x)
- 애너테이션에도 인터페이스처럼 `상수`를 정의할 수 있지만, `디폴트 메서드`는 정의할 수 없다.
```java
// 애너테이션 정의
@interface TestInfo {
    int count();
    String testedBy();
    String[] testTools();
    TestType testType();    // enum TestType { FIRST, FINAL }
    DateTime testDate();    // 자신이 아닌 다른 애너테이션을 포함할 수 있다.
}

// 애너테이션 사용
@TestInfo(
    count = 3, testedBy = "KIM",
    testTools = {"JUnit", "AutoTester"},
    testType = TestType.FIRST,
    testDate = @DateTime(yymmdd = "160101", hhmmss = "235959")
)
public class NewClass { ... }
```

### 5.1 애너테이션의 요소
- 적용시 값을 지정하지 않으면, 사용될 수 있는 기본값 지정가능 (null은 기본값으로 지정 x)
```java
@interface TestInfo {
    // 기본값 1로 지정
    int count() default 1;
}

@TestInfo   // @TestInfo(count = 1)과 동일하다.
public class NewClass { ... }
```

- 요소가 하나이고 이름이 `value`일 때는 요소의 이름 생략 가능
```java
@interface TestInfo {
    String value();
}

@TestInfo("passed") // @TestInfo(value = "passed")와 동일
class NewClass { ... }
```

- 요소의 타입이 배열인 경우, 괄호{}를 사용해야 한다
```java
@interface TestInfo {
    String[] testTolls();
}

// 이너테이션 사용
@Test(testTools = {"JUnit", "AutoTester"})  // <1>
@Test(testTools = "JUnit")  // <2>
@Test(testTools = {})   // <3>, 값이 없을 때는 괄호{}를 반드시 사용한다.
```

### 5.2 모든 애너테이션의 조상
- `java.lang.annotation.Annotation`
- Annotation은 모든 애너테이션의 조상이지만 `상속은 불가` (다른 애너테이션들이 Annotation의 추상메서드를 포함하고 있는 셈)
- Annotation은 `인터페이스`이다.
- Annotation인터페이스안의 메서드는 추상메서드이며 컴파일러가 대신 구현을 해주기때문에 구현하지 않고 사용가능하다.
```java
package jave.lang.annotation;

public interface Annotation {
    boolean equals(Object obj);
    int hashCode();
    String toString();

    Class<? extends Annotation> annotationType();
}
```

## 6. 마커 애너테이션
- Marker Annotation
- 요소가 하나도 정의되지 않은 애너테이션
- 애너테이션이 붙이있는 것만으로도 애너테이션을 사용하는 프로그램에게 충분한 정보를 제공
```java
@Target(ElementType.METHOD)
public @interface Test {}   // 마커 애너테이션
// 마커 애너테이션 사용
@Test   // 요소가 없다.
public void method() { }
```

## 7. 애너테이션 요소의 규칙
- 애너테이션의 요소를 선언할 때 아래의 규칙을 반드시 지켜야 한다.
    + 요소의 타입은 `기본형`, `String`, `enum`, `애너테이션`, `Class`만 허용
    + 괄호()안에 `매개변수`를 선언할 수 없다.
    + `예외`를 선언할 수 없다.
    + 요소를 `타입 매개변수`로 정의할 수 없다.

# Reference
> 자바의 정석 - 남궁성

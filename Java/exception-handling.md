# 예외처리

## 1. 프로그램 오류
- 컴파일 에러, 런타임 에러, 논리적 에러
- 에러 : 심각한 오류(어쩔 수 없다), 예외 : 미약한 오류(처리 가능)

## 2. 예외처리의 정의와 목적
- 정의 : 예외의 발생을 대비한 코드 작성
- 목적 : 프로그램의 비정상 종료를 막고 정상적인 실행상태 유지

## 3. 직접 처리 (try-catch)
```java
try {
  // 예외가 발생할 가능성이 있는 문장
} catch (Exception1) {
  // Exception1이 발생했을 경우, 이를 처리하기위한 문장
} catch (Exception2) {
  // Exception2이 발생했을 경우, 이를 처리하기위한 문장
...
```

## 4. try-catch 흐름
1. try블럭에서 예외가 발생한 경우
- 일치하는 예외가 있는지 catch 확인
- 일치하면 catch 내의 문장 수행, 일치하지 않으면 예외처리 x(비정상 종료)

2. try블럭에서 예외가 발생하지 않은 경우
- catch안거치고 try-catch빠져나와 이어서 수행

## 5. 예외 클래스의 계층구조
- RuntimeException클래스와 그 자손들   
-> 프로그래머의 실수로 발생하는 예외 (예외처리 선택)
- Exception클래스와 그 자손들 (RuntimeException클래스도 자손이자만 RuntimeException클래스를 제외한 클래스를 의미)   
-> 외적인 요인에 의해 발생하는 예외 (예외처리 필수)

## 6. 예외 발생과 catch 블럭
- Exception클래스는 예외의 최고 조상으로 catch블록을 이용해 모든 종류의 예외를 처리할 수 있다. (단 catch문의 맨 마지막에 사용해야한다)
- 발생한 예외 객체를 catch블럭의 참조변수로 접근할 수 있다.
```java
...
} catch (AritmethicException ae) {
  // 예외발생 당시 호출 스택에 있는 메서드의 정보와 예외 메서지를 화면에 출력
  ae.printStackTrace(); 
  // 발생한 예외클래스의 인스턴스에 저장된 메시지를 얻을 수 있다.
  System.out.println("예외 메서지 : " + ae.getMessage());  
  }
```
## 7. 멀티 catch 블럭
- 내용이 같은 catch블럭을 하나로 합친것 (jdk 1.7부터)
- 상속관계의 예외클래스에서는 사용하지않는다 (사실 사용할 필요도 없다)
```java
try { ... 
} catch (ExceptionA | ExceptionB e) {
  e.printStackTrace();
}
```

## 8. 예외 발생시키기
- 먼저, 연산자 new를 이용해 예외 클래스의 객체 생성한 후
- throw를 이용해 예외를 발생시킨다.
```java
try {
    Exception e = new Exxception("고의로 발생");
    throw e;  // 예외 발생
} catch (Exception e) {
  ...
}
```

## 9. checked 예외, unchecked 예외
- checked 예외 : 컴파일러가 예외처리 여부를 체크 (예외처리 필수) <- Exception과 그 자손들
- unchecked 예외 : 컴파일러가 예외처리 여부를 체크 안함 (예외처리 선택) <- RuntimeException과 그 자손들

## 10. finally 블럭
- 예외의 발생여부와 관계없이 실행되어야할 코드가 있으면 finally 블럭안에 넣는다.
- 순서 : try -> catch -> finally (무조건 마지막에 !!)

## 11. 메서드의 예외 선언
- 예외처리를 직접하는 것이 아닌 호출한 메서드 쪽에서 예외처리를 하라고 떠넘기는 것이다.
```java
void method() throws Exception1, Exception2, Exception3, ... {
  // 메서드 내용
}
```

## 12. 예외 되던지기(re-throwing)
- 예외를 처리한 후 다시 예외를 생성해서 호출한 메서드로 전달하는 것
- 즉, 예외처리를 2번사용해서 첫번째에 일부 해결하고 해결할 수 없는 부분을 throw로 던져서 예외처리를 실행해서 예외를 해결 (try-catch 2번)

## 13. 사용자정의 예외 만들기
- 기존의 예외 클래스를 상속받아와 새로운 예외 클래스를 정의하는 것
- 과정 : Exception or RuntimeException 중 상속받을 예외 클래스 선택 -> 문자열을 매개변수로 받는 생성자와 조상인 Exception클래스 생성자 호출
> tip : RumtimeException클래스는 예외처리가 선택이기 때문에 RuntimeException을 사용하도록 하자 (꼭! 필요한 경우 Exception 사용)
```java
class MyException extends Exception {
  // 문자열을 매개변수로 받는 생성자
  MyException(String msg) {
  // 조상 Exception클래스의 생성자 호출
  super(msg);
  }
}
```

## 14. 연결된 예외
- 예외안에 예외를 넣는 것 (한 예외가 다른 예외를 발생시킬 수 있다.)
- 여러 예외를 큰 분류의 예외로 묶고 싶을 때 사용
- 필수 예외를 선택 예외로 바꿀 때 사용 (Exception -> RuntimeException)
```java
// 지정한 예외를 원인 예외로 등록
Throwable initCause(throwable cause)  
// 원인예외 반환
Throwable getCause()
```

```java
public class Throwable implements Serializable {
  ...
  // 원인예외를 저장하기 위한 iv인 cause (3)
  private Throwable cause = this;
  ...
  // 지정한 예외를 원인예외로 지정 (1)
  public synchronized Throwable initCause(Throwable cause) {
    ...
    // this.cuase를 원인예외로 등록 (2)
    this.cause = cause;
    return this;
  }
}
```

# Reference
> 자바의 정석 - 남궁성

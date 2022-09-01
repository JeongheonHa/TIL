# Lambda

## 1. 람다식
- 함수(메서드)를 간단한 '식'으로 표현하는 방법
- 익명 함수 (정확히는 익명 객체)
- 함수와 메서드의 차이
    + 근본적으로 동일, 함수는 일반적 용어, 메서드는 객체지향개념 용어
    + 함수는 클래스에 독립적, 메서드는 클래스에 종속적

### 1.1 람다식 작성하기
1. 메서드의 이름과 반환타입을 제거하고 '->'를 블력{}앞에 추가한다.
2. 반환 값이 있는 경우, 식이나 값만 적고 return문 생략 가능 (끝에 ; 안붙임)
3. 매개변수의 타입이 추론 가능하면 생략가능(대부분의 경우 생략가능)

### 1.2 주의사항
1. 매개변수가 하나인 경우, 괄호() 생략가능(타입이 없을 때만)
2. 블록 안의 문장이 하나뿐 일 때, 괄호{} 생략가능(끝에 ; 안붙임) </br> 
단, 하나뿐인 문장이 return문이면 괄호{} 생략불가

## 2. 함수형 인터페이스
- 단 하나의 추상 메서드만 선언된 인터페이스
- 람다식을 다루기 위해 사용
- 함수형 인터페이스 타입의 참조변수로 람다식을 참조할 수 있다.
```java
interface MyFunction {
    public abstract int max(int a, int b);
}
// 익명 객체 생성
Myfunction f = new MyFunction() {
    public int max(int a, int b) {
        return a > b ? a : b;
    }
};
// 람다식
MyFunction f = (a, b) -> a > b ? a : b;
int value = f.max(3.5);
```

### 2.1 함수형 인터페이스 타입의 매개변수, 반환타입
- 함수형 인터페이스 타입의 매개변수
```java
void aMethod(MyFunction f) {    // 메서드의 매개변수로 람다식을 받겠다는 의미
    f.myMethod();   // MyFunction에 정의된 메서드 호출 (람다식 호출)
}
// 매개변수로 람다식을 받는 메서드
Myfunction f = () -> System.out.println("myyMethod()");
aMethod(f);
// 같은 의미
aMethod(() -> System.out.println("myMethod()"));
```

- 함수형 인터페이스 타입의 반환타입
```java
// 람다식 반환
MyFunction myMethod() {
    MyFunction f = () -> {};
    return f;
}
// 같은 의미
MyFunction myMethod() {
    return () -> {};
}
```

# Reference
> 자바의 정석 - 남궁성

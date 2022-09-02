# Lambda

## 1. 람다식
- 함수(메서드)를 간단한 `'식'`으로 표현하는 방법
- `익명 함수` (정확히는 익명 객체)
- 함수와 메서드의 차이
    + 근본적으로 동일, 함수는 일반적 용어, 메서드는 객체지향개념 용어
    + 함수는 클래스에 독립적, 메서드는 클래스에 종속적

### 1.1 람다식 작성하기
1. 메서드의 이름과 반환타입을 제거하고 `'->'`를 블력{}앞에 추가한다.
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
// 함수형 인터페이스
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
// 함수형 인터페이스 타입의 참조변수로 람다식 참조
MyFunction f = (a, b) -> a > b ? a : b;
int value = f.max(3.5);
```

### 2.1 함수형 인터페이스 타입의 매개변수, 반환타입
- 함수형 인터페이스 타입의 매개변수
```java
// 함수형 인터페이스
@FunctionalInterface
interface MyFunction {
    void myMethod();
}

void aMethod(MyFunction f) {    // 메서드의 매개변수로 람다식을 받겠다는 의미
    f.myMethod();   // MyFunction인터페이스의 myMethod()를 구현한 객체의 myMethod()를 수행 (람다식 호출)
}
// 익명 객체(람다식)에 함수형 인터페이스의 참조변수로 참조
Myfunction f = () -> System.out.println("myMethod()");
aMethod(f);
// 같은 의미
aMethod(() -> System.out.println("myMethod()"));
```

- 함수형 인터페이스 타입의 반환타입   
: 변수 처럼 메서드를 주고받는 것이 가능해진다. (사실상 객체를 주고받는 것이므로 달리지는 것은 없다.)
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

### 2.2 람다식의 타입과 형변환
- 함수형 인터페이스로 람다식을 참조할 수 있는 것일 뿐, 람다식의 타입이 함수형 인터페이스의 타입과 일치하는 것은 아니다.
- 람다식은 익명 객체이고 익명 객체는 타입이 없다. (정확히는 있지만 컴파일러가 임의로 정하기 때문에 알 수 없다.)
- 형변환 생략가능
- Object 타입으로 형변환 x, 오직 함수형 인터페이스로만 형변환 가능
```java
MyFunction f = (MyFunction) (() -> {});
```

## 3. java.util.function 패키지
- 자주 쓰이는 형식의 메서드를 함수형 인터페이스로 미리 정의해놓은 패키지

| 함수형 이터페이스 | 메서드 | 설명 |
|---|---|---|
| java.lang.Runnable | void run() | 매개변수 x, 반환값 x |
| Supplier<T> | T get() | 매개변수 x, 반환값 o |
| Consumer<T> | void accept(T t) | 매개변수 o, 반환값 x |
| Function<T,R> | R apply(T t) | 일반적인 함수, 매개변수 o, 반환값 o |
| Predicate<T> | boolean test(T t) | 조건식을 표현하는데 사용 </br> 매개변수 o, 반환 값 o (boolean) |

### 3.1 매개변수가 두 개인 함수형 인터페이스

| 함수형 인터페이스 | 메서드 |
|---|---|
| BiConsumer<T,U> | void accept(T t, U u) | 
| BiPredicate<T,U> | boolean test(T t, U u) |
| BiFunction<T,U,R> | R apply(T t, U u) |
> Supplier는 매개변수는 없고 반환값만 존재하는데, 메서드는 두 개의 값을 반환할 수 없기때문에 BiSupplier가 없다.   
> 두 개 이상의 매개변수를 갖는 함수형 인터페이스가 필요한 경우 직접 만들어야한다.

### 3.2 UnaryOperator & BinaryOperator
- Function의 변경으로 `매개변수의 타입`과 `반환타입의 타입`이 모두 `일치`한다는 점을 제외하고 Function과 동일
- 각각 `Function`과 `BiFunction`의 자손이다.

| 함수형 인터페이스 | 메서드 |
|---|---|
| UnaryOperator<T> | T apply(T t) |
| BinaryOperator<T>| T apply(T t, T t) |

### 3.3 컬렉션 프레임웍과 함수형 인터페이스

| 인터페이스 | 메서드 | 설명 |
|---|---|---|
| Collection | boolean removeIf(Predicate<E> filter) | 조건에 맞는 요소를 삭제 |
| List | void replaceAll(UnaryOperator<E> operator) | 모든 요소를 변환하여 대체 |
| Iterable | void forEach(Consumer<T> action) | 모든 요소에 작업 action을 수행 |
| Map | V compute(K key, BiFunction<K,V,V> f) | 지정된 키의 값에 작업 f를 수행 |
| | V computeIfAbsent(K key, Function<K,V> f) | 키가 없으면 작업 f 수행 후 추가 |
| | V computeIfPresent(K key, BiFunction<V,V,V> f) | 지정된 키가 있을 때 작업 f 수행 |
| | V merge(K key, V value, BiFunction<V,V,V> f) | 모든 요소에 병합작업 f를 수행 | 
| | void forEach(BiConsumer<K,V> action) | 모든 요소에 작업 action을 수행 |
| | void replaceAll(BiFunction<K,V,V> f) | 모든 요소에 치환작업 f를 수행 |

### 3.4 기본형을 사용하는 함수형 인터페이스
- 기본형을 사용하기 때문에 지네릭 타입나 래퍼 클래스를 사용하는 것보다 효율적이다.

| 함수형 인터페이스 | 메서드 | 설명 |
|---|---|---|
| `Double`To`Int`Function | int applyAsInt(double d) |`A`To`B`Function은 입력이 A타입출력이 B타입 |
| To`Int`Function<T> | int applyAsint(T value) | To`B`Function은 입력이 지네릭 타입, 출력이 B타입이다. |
| `Int`Function<R> | R apply(T t, U u) | `A`Function은 입력이 A타입이고 출력은 지네릭 타입이다. |
| Obj`Int`Consumer<T> | void accept(T t, U u) | Obj`A`Function은 입력이 T, A타입이고 출력은 없다. |

# Reference
> 자바의 정석 - 남궁성

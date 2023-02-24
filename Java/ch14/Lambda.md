# Lambda

## 1. 람다식
- 함수(메서드)를 간단한 `'식'`으로 표현하는 방법
- `익명 함수` (정확히는 익명클래스의 객체)
- 함수와 메서드의 차이
    + 근본적으로 동일, 함수는 일반적 용어, 메서드는 객체지향개념 용어
    + 함수는 클래스에 독립적, 메서드는 클래스에 종속적

### 1.1 람다식 작성하기
1. 메서드의 이름과 반환타입을 제거하고 `'->'`를 블럭`{}`앞에 추가한다.
2. 반환 값이 있는 경우, 식이나 값만 적고 return문 생략 가능 (끝에 ; 안붙임)
3. 매개변수의 타입이 추론 가능하면 생략가능(대부분의 경우 생략가능)

### 1.2 주의사항
1. 매개변수가 하나인 경우, 괄호(`()`) 생략가능(타입이 없을 때만)
2. 블록 안의 문장이 하나뿐 일 때, 괄호(`{}`) 생략가능(끝에 `;` 안붙임) </br> 
단, 하나뿐인 문장이 return문이면 괄호(`{}`) 생략불가


## 2. 함수형 인터페이스
- `하나`의 추상 메서드만 선언된 인터페이스
- 람다식을 다루기 위해 사용
- 함수형 인터페이스 타입의 참조변수로 람다식을 참조할 수 있다. (참조변수의 타입은 클래스 or 인터페이스)
- `static메서드`와 `default메서드`의 개수는 제약이 없다.
- `@FunctionalInterface`를 붙이면 컴파일러가 함수형 인터페이스를 올바르게 정의했는지 확인해준다.

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
: 이 메서드를 호출할 때 람다식을 참조하는 참조변수를 매개변수로 지정해야한다는 뜻
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

### 2.3 외부 변수를 참조하는 람다식
- 람다식에서 외부에 선언된 변수에 접근하는 규칙은 익명 클래스와 동일
- 람다식 내에서 참조하는 지역변수는 final이 붙지 않아도 상수로 간주
- 외부 지역변수와 같은 이름의 람다식 매개변수는 혀용되지 않는다.
```java
@FunctionalInterface
interface Function {
    void myMethod();
}

class Outer {
    int val = 10;

    class Inner {
        int val = 20;

        void method(int i) {    // void method(final int i)와 같다.
            int val = 30;

            MyFunction f = () -> {  // MyFunction f = (i) -> , error
                System.out.println("i : " + i);
                System.out.println("val : " + val);
                System.out.println("this.val : " + this.val);
                System.out.println("Outer.this.val : " + Outer.this.val);
            }
            f.myMethod();
        }
    }
}

class LambdaEx {
    public static void main(String[] args) {
        Outer outer = new Outer();
        Outer.Inner inner = outer.new Inner();
        inner.method(100);
    }
}

/*
i : 100
val : 30
this.val : 20
Outer.this.val : 10
```

## 3. java.util.function 패키지
- 자주 쓰이는 형식의 메서드를 함수형 인터페이스로 미리 정의해놓은 패키지

| 함수형 이터페이스 | 메서드 | 설명 |
|---|---|---|
| java.lang.Runnable | void run() | 매개변수 x, 반환값 x |
| Supplier`<T>` | T get() | 매개변수 x, 반환값 o |
| Consumer`<T>` | void accept(T t) | 매개변수 o, 반환값 x |
| Function`<T,R>` | R apply(T t) | 일반적인 함수, 매개변수 o, 반환값 o |
| Predicate`<T>` | boolean test(T t) | 조건식을 표현하는데 사용 </br> 매개변수 o, 반환 값 o (boolean) |
> Predicate는 Function의 변형으로 반환타입이 boolean이라는 것만 다르다. (조건식을 람다식으로 표현하는데 사용)

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
| UnaryOperator`<T>` | T apply(T t) |
| BinaryOperator`<T>`| T apply(T t, T t) |

### 3.3 컬렉션 프레임웍과 함수형 인터페이스
- 컬렉션 프레임웍의 인터페이스에 함수형 인터페이스를 사용하는 디폴트 메서드 추가

| 인터페이스 | 메서드 | 설명 |
|---|---|---|
| Collection | boolean removeIf(Predicate`<E>` filter) | 조건에 맞는 요소를 삭제 |
| List | void replaceAll(UnaryOperator`<E>` operator) | 모든 요소를 변환하여 대체 |
| Iterable | void forEach(Consumer`<T>` action) | 모든 요소에 작업 action을 수행 |
| Map | V compute(K key, BiFunction<K,V,V> f) | 지정된 키의 값에 작업 f를 수행 |
| | V computeIfAbsent(K key, Function<K,V> f) | 키가 없으면 작업 f 수행 후 추가 |
| | V computeIfPresent(K key, BiFunction<K,V,V> f) | 지정된 키가 있을 때 작업 f 수행 |
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

## 4. Function의 합성과 Predicate의 결합
- java.util.function패키지의 함수형 인터페이스에 정의되어있는 `default메서드`와 `static메서드`
- Function과 Perdicate에 정의된 메서드에서만 살펴보지만, 다른 함수형 인터페이스의 메서드도 유사하다.
```java
// Function
default <V> Function<T.V> andThen(Function<? super R,? extends V> after)
default <V> Function<V,R> compose(Function<? super V,? extends T> before)
static  <T> Function<T,T> indentity()   // Function인터페이스는 두 타입이 같아도 반드시 두개의 타입을 지정해줘야한다.

// Predicate
default Predicate<T>   and(Predicate<? super T> other)
default Perdicate<T>   or(Predicate<? super T> other)
default Perdicate<T>   negate()
static <T> Predicate<T> isEqual(Object targetRef)
```

### 4.1 Function의 합성
- 두 람다식을 합성하여 새로운 람다식을 만든다. (like. 수학에서 함수의 합성)
```java
// andthen()
default <V> Function<T,V> andThen(Function<? super R, ? extends V> after)
// compose()
default <V> Function<V,R> compose(Function<? super V, ? extends T> before)

// 예시
// andThen()
Function<String, Integer> f = (s) -> Integer.parseInt(s, 16);
Function<Integer, String> g = (i) -> Integer.toBinaryString(i);
Function<String, String> h = f.andThen(g);  // String받아서 String 반환
// compose()
Function<Integer, String> g = (i) -> Integer.toBinaryString(i);
Function<String, Integer> f = (s) -> Integer.parseInt(s, 16);
Function<Integer, Integer> h = f.compose(g);
```

### 4.2 Predicate의 결합
- `and()`, `or()`, `negate()`로 연결해서 하나의 새로운 Predicate로 결합
```java
Predicate<Integer> p = i -> i < 100;
Predicate<Integer> q = i -> i < 200;
Predicate<Integer> r = i -> i%2 == 0;
Predicate<Integer> notP = p.negate();

Predicate<Integer> all = notP.and(q.or(r));
System.out.println(all.test(150));
// 람다식을 직접 넣을 경우
Predicate<Integer> all = notP.and(i -> i < 200).or(i -> i%2 == 0);  // 같은 의미
```
> Predicate의 끝에 negate()를 붙이면 조건식 전체가 부정이 된다.

## 5. 메서드 참조
- 람다식이 하나의 메서드만 호출하는 경우 `메서드 참조`라는 방법으로 람다식을 더 간략히 할 수 있다.
- 메서드 참조는 람다식을 마치 static 변수처럼 다룰 수 있게 해준다.

| 종류 | 람다 | 메서드 참조 |
|---|---|---|
| static 메서드 참조 | (x) -> ClassName.method(x) | ClassName::method |
| 인스턴스 메서드 참조 | (obj, x) -> obj.method(x) | ClassName::method |
| 특정 객체 인스턴스 메서드 참조 | (x) -> obj.method(x) | obj::method |

```java
// static 메서드 참조
Function<String, Integer> f = (String s) -> Integer.parseInt(s);
Function<String, Integer> f= Integer::parseInt;

// 인스턴스 메서드 참조
BiFunction<String, String, Boolean> f= (s1, s2) -> s1.equals(s2);
BiFunction<String, String, Boolean> f = String::equals; 
// 두개의 String을 받아 Boolean으로 반환하는 equals가 다른 클래스에도 존재할 수 있기 때문에 equals앞에 클래스이름을 반드시 붙여준다.

// 특정 객체 인스턴스 메서드 참조
MyClass obj = new MyClass();
Function<String, Boolean> f = (x) -> obj.equals(x);
Function<String, Boolean> f2 = obj::equals;  
```

### 5.1 생성자의 메서드 참조
- 생성자를 호출하는 람다식을 메서드 참조로 변환
```java
Supplier<MyClass> s = () -> new MyClass();
Supplier<MyClass> s= MyClass::new;
// 매개변수가 있는 생성자
Function<Integer, MyClass> f = (i) -> new MyClass(i);
Function<Integer, MyClass> f2 = MyClass::new;

BiFunction<Integer, String, MyClass> bf = (i, s) -> new MyClass(i, s);
BiFunction<Integer, String, MyClass> bf2 = MyClass::new;

// 배열을 생성할 경우
Function<Integer, int[]> f = x -> new int[x];
Function<Integer, int[]> f2 = int[]::new;
```
# Reference
> 자바의 정석 - 남궁성

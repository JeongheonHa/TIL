# 열거형(enum)

## 1. 열거형
- 관련된 상수들을 같이 묶어 놓은 것
- Java는 타입에 안전한 열거형을 제공한다.
- `열거형 상수`는 하나하나가 `객체`이다.

```java
class Card {
    static final int CLOVER = 0;
    static final int HEART = 1;
    static final int DIAMOND = 2;
    static final int SPADE = 3;

    static final int TWO = 0;
    static final int THREE = 1;
    static final int FOUR = 2;

    final int kind;
    final int num;
}
if(Card.CLOVER == Card.TWO) {}  // true, 상수의 값만을 비교하기 때문에 

// 열거형
class Card {
    // 열거형 Kind 정의
    enum Kind { CLOVER, HEART, DIAMOND, SPADE }
    // 열거형 Value 정의
    enum Value { TWO, THREE, FOUR }

    final Kind kind;
    final Value value;
}
if(Card.Kind.CLOVER == Card.Value.TWO) {}   // false, 값 뿐만아니라 타입까지 비교
```

## 2. 열거형의 정의

- 열거형 정의하는 방법

```java
enum 열거형이름 { 상수명1, 상수명2, ... }
enum Direction { EAST, SOUTH, WEST, NORTH }
```

- 열거형 타입의 변수를 선언하고 사용하는 방법
```java
class Unit {
    int x, y;
    // 열거형 인스턴스 변수를 선언
    Direction dir;
    // 방향을 EAST로 초기화
    void init() {
        dir = Direction.EAST;
    }
}
```

- 열거형 상수의 비교
: `==`, `compareTo()` 사용 (`equals()`나 `'<'`,`'>'` 사용 x)
```java
if(dir == Direction.EAST) { // ok
    x++;
} else if (dir > Direction.WEST) {  // error
    ...
} else if (dir.compareTo(Direction.WEST) > 0) { // ok
    ...
}
```

- `switch문`의 조건식에도 열거형 사용 가능
```java
void move() {
    switch(dir) {
        case EAST: x++; // Direction.EAST라고 쓰면 안된다.
        break;
        case WEST: x--;
        break;
        case SOUTH: y++;
        break;
        case NORTH: y--;
        break;
    }
}
```

## 3. 열거형의 조상
- `java.lang.Enum`
- 모든 열거형은 Enum클래스의 자손이며 아래의 메서드를 상속받는다.

| 메서드 | 설명 |
|---|---|
| Class`<E>` getDeclaringClass() | 열거형의 `Class객체 반환` |
| String name() | 열거형 상수의 이름을 `문자열`로 `반환` |
| int ordinal() | 열거형 상수가 정의된 `순서 반환` (0부터) |
| T valueOf(Class`<T>` enumType, String name) | 지정된 열거형에서 name과 일치하는 `열거형 상수 반환` |

- `values()`, `valueOf()`는 컴파일러가 자동으로 추가
    + `values()` : 열거형 상수가 가지고 있는 `모든 상수`를 `배열`로 `반환`
    + `valueOf(String s)` : 열거형 상수 이름을 `문자열`로 주면 `열거형 상수`를 `반환`
```java
// 정의
static E[] values()
static E valueOf(String name)
// 사용
Direction[] dArr = Direction.values();
Direction d = Direction.valueOf("WEST");
Dircetion d = Direction.WEST;   // valueOf와 같은 의미
```

## 4. 열거형에 멤버 추가

- 불연속적인 열거형 상수의 경우, `원하는 값`을 `괄호()`안에 적는다.
- 열거형 상수 마지막에 `';'`를 붙인다.

```java
enum Direction { EAST(1), SOUTH(5), WEST(-1), NORTH(10); }   // 사실상 생성자 호출
```

- `괄호()`를 사용하려면, `인스턴스 변수`와 `생성자`를 새로 `추가`해야한다.
- `열거형 생성자`는 암묵적으로 `private`이기 때문에 다른 클래스에서 new를 이용해 생성자를 호출할 수 없다. (다른 클래스에서 객체 생성 x)
- 열겨형 인스턴스 변수는 반드시 final을 붙일 필요는 없지만 상수의 값을 저장하기 위한 변수이기 때문에 final을 붙여준다.
```java
enum Direction {
    EAST(1, ">"), SOUTH(5, "V"), WEST(-1, "<"), NORTH(10, "^");
    // 정수를 저장할 필드(인스턴스 변수) 추가
    private final int value;    // 값의 개수만큼 iv 선언
    private final String symbol;
    // 값을 받을 수 있는 생성자 추가
    Direction(int value, String symbol) { 
        this.value = value;
        this.symbol = symbol; 
    }    // `private` Direction(int value) { this.value = value; }

    public int getValue() { return value; }
    public String getSymbol() { return symbol; }
}

Direction d = new Direction(1, ">"); // error
```

## 5. 열거형에 추상메서드 추가
- 열거형 상수에 주어진 `값(괄호)`으로 다른 결과 값을 얻고싶을 때 열거형에 `추상 메서드`를 선언하여 각 열거형 상수가 `구현`하도록한다.
- 인스턴스 변수의 제어자를 `protected`로 해야 각 열겨형 상수에 접근할 수 있다.
```java
enum Transportation {
    // 열거형 상수에 추상 메서드 구현 (익명 클래스와 유사)
    BUS(100) { int fare(int distance) { return distance * BAISC_FARE; } },
    TRAIN(150) { int fare(int distance) { return distance * BASIC_FARE; } },
    SHIP(100) { int fare(int distance) { return distance * BASIC_FARE; } },
    AIRPLANE(300) { int fare(int distance) { return distance * BASIC_FARE; } };
    
    protected final int BASIC_FARE; // protected로 해야 각 상수에서 접근가능

    Transportation (int basicFare) {
        BASIC_FARE = basicFare;
    }
    public int getBasicFare() { return BASIC_FARE; }
    // 추상 메서드 선언
    abstract int fare(int distance);    // 거리에 따른 요금 계산
}

class Ex {
    public static void main(String[] args) {
        System.out.println("bus fare = " + Transportation.BUS.fare(100));   // distance = 100
        System.out.println("train fare = " + Transportation.TRAIN.fare(100));
        System.out.println("ship fare = " + Transportation.SHIP.fare(100));
        System.out.println("airplane fare = " + Transportation.AIRPLANE.fare(100));
    }
}
```

# Reference
> 자바의 정석 - 남궁성

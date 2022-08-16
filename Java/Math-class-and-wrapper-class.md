# Math클래스 & wrapper클래스

## 1. Math클래스의 메서드
- 수학계산에 유용한 메서드로 구성되어 있다. (모두 static 메서드이다.)

- `static 기본형 타입 abs(기본형 매개변수)`   
: 주어진 값의 절대값을 반환

- `static double ceil(double a)`   
: 주어진 값을 올림하여 반환

- `static double floor(double a)`   
: 주어진 값을 버림하여 반환

- `static 기본형 타입 max (기본형 매개변수, 기본형 매개변수)`   
: 주어진 두 값을 비교하여 큰 쪽을 반환

- `static 기본형 타입 min (기본형 매개변수, 기본형 매개변수)`   
: 주어진 두 값을 비교하여 작은 쪽을 반환

- `static double random()`   
: 0.0 <= x < 1.0범위의 임의의 double값을 반환

- `static double rint(double a)`   
: 짝수 0.5일 때 반올림 x, 홀수 0.5일 때 반올림 o (오차를 줄일 수 있지만 잘 쓰이진 않는다.) 

- `static long round(double/float a)`   
: 소수점 첫번째 자리에서 반올림한 정수 값(long)반환

## 2. wrapper클래스
- 기본형을 감싸는 클래스
- 기본형 값을 객체로 다뤄져야할 때 사용한다.
- 내부적으로 기본형 변수를 가지고 있다.
- equals가 오버라이딩 되어있다. (객체 비교 -> 값 비교)
```java
public final class Integer extends Number implements Comparable {
    ...
    private int value;
}
```
| 기본형 | boolean | char | byte | short | int | long | float | double |
|---|---|---|---|---|---|---|---|---|
|wrapper 클래스 | Boolean | Character | Byte | Short | Integer | Long | Float | Double |

> 기본형 값이 객체로 만들어지지 않은 이유는 성능 때문이다.

## 3. Number클래스
- 숫자를 멤버변수로 갖는 클래스의 조상 (추상클래스)
- Byte, Short, Integer, Long(10^19), Float, Double(10^308, 정밀도 15자리), BigInteger, BigDecimal
```java
public abstract class Number implements java.io.Serializalbe {
    public abstract int intValue();
    public abstract long longValue();
    public abstract float floatValue();
    public abstract double doubleValue();

    public byte byteValue() {
        return (byte)intValue();
    }

    public short shortValue() {
        return (short)intValue();
    }
}
```
> 래퍼 객체의 값을 기본형으로 바꿔주는 메서드들을 가지고 있다.   
> Long의 범위를 넘길거 같으면 BigInteger, Double의 범위와 정밀도를 넘길거 같으면 BigDecimal

## 4. 문자열을 숫자로 변환하는 3가지 방법
- `int i = new Integer("100").intValue();`(1)
- `int i2 = Integer.parseInt("100");` (2)
- `Integer i3 = Integer.valueOf("100");` (3) // Integer 대신 int 가능 (autoboxing)   

## 5. n진법의 문자열을 숫자로 변환
```java
int i4 = Integer.parseInt("100", 2);    // 100(2) -> 4
int i5 = Integer.parseInt("100", 8);    // 100(8) -> 64
int i6 = Integer.parseInt("100", 16);   // 100(16) -> 256
int i7 = Integer.parseInt("FF", 16);    // FF(16) -> 255
```

# Reference
> 자바의 정식 - 남궁성

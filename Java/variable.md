# variable

## 1. 변수의 명명규칙
- 대소문자가 구분되며 길이에 제한이 없다.
- 예약어를 사용해서는 안된다.
- 숫자로 시작해서는 안된다.
- 특수문자는 '_'와 '$'만을 허용한다.

## 2. 변수의 타입
- `값`은 `문자`와 `숫자`로 나누며 숫자는 `정수`와 `실수`로 나눈다.
- 참조형 변수 간의 연산은 할 수 없다.
- 기본형 : 논리형, 문자형, 정수형, 실수형
- 참조형 : 기본형 제외 나머지 타입, 객체의 주소 저장

### 2.1 기본형
- boolean을 제외한 나머지 7개의 기본형은 서로 연산과 변환이 가능
- char 은 2byte
- int는 CPU가 가장 효율적으로 처리할 수 있는 타입
- float 정밀도 : 소수점 6자리, double 정밀도 : 소수점 14자리

### 2.2 상수와 리터럴
- 상수는 선언과 동시에 초기화한다.
- 변수의 타입은 저장될 값의 타입(리터럴의 타입)에 의해 결정
```java
String str = "";    // 빈 문자열 가능
char ch = ' ';      // 빈 문자열 x
```
- 문자열과 뎃셈연산을 수행하면 그 결과는 문자열

### 2.3 형식화된 출력
- printf()
- %b, %d, %o, %x(%X), %f, %e(%E), %c, %s
- `%f`는 소수점 아래 7자리에서 `반올림`

### 2.4 Scanner
- 화면으로부터 입력받는 방법
```java
import java.util.*;

Scanner scanner = new Scanner(System.in);

String input = scanner.nextLine();  // 입력한 내용 문자열로 반환
```

## 3. 진법
- bit : 컴퓨터가 값을 저장할 수 있는 최소단위
- byte : 데이터의 기본 단위
- word : CPU가 한 번에 처리할 수 있는 데이터의 크기

### 3.1 실수의 진법 변환
- 10진 -> 2진
    + 소수부에 2 곱한 후 0.에 정수부 나열
- 2진 -> 10진
    + 2진 소수부에 2^-1 ... 곱해서 더한다.

### 3.2 2의 보수
- 음수의 절대값을 2진수로 변환
- 2진수의 1을 0으로 0은 1로 바꾼다. (1의 보수)
_ 결과에 1을 더한다.

## 4. 기본형
- boolean : false가 기본값
- char : 문자의 유니코드(정수)가 저장
- 인코딩 : 문자 -> 코드, 디코딩 : 코드 -> 문자    

### 4.1 정수형
- JVM의 피연산자 스택이 피연산자를 4 byte단위로 저장하기 때문에 크기가 4 byte보다 작은 자료형의 값을 계산할 때는 4 byte로 변환하여 연산이 수행 (int를 사용하는 것이 효율적이다.)
- overflow : 타입이 표현할 수 있는 값의 범위를 넘어서는 것

### 4.2 실수형
- overflow발생시 무한대가 된다.
- underflow : 최소값보다 작은 값이 되는 경우, 0이된다.

### 4.3 실수형의 저장방식
- float : 1(S) + 8(E) + 23(M)
- double : 1(S) + 11(E) + 52(M)
- S : 부호, E : 지수, M : 가수
- 지수의 범위는 '-126 ~ 127'이다. (-127,128은 양의 무한대, 음의 무한대 등의 특별한 값을 표현하기위해 예약)
- 기저법 : 지수를 저장하는 방식 (지수에 127을 더해서 저장하고 사용할 때 127을 뺀다.)

## 5. 형변환
- 변수 or 상수의 타입을 다른 타입으로 변환하는 것
- 기본형과 참조형간의 형변환 x
- 정수형 간의 형변환
    + 값 손실 주의
- 실수형 간의 형변환
    + 가수의 24번째에서 반올림이 발생할 수 있다.
    + float타입의 범위를 넘는 값을 float으로 형변환하는 경우 무한대, 0 값을 얻는다.
- 정수형을 실수형으로 형변환
    + 정수형 10진수로 7자리 이상은 float이 아닌 double을 이용
- 실수형을 정수형으로 형변환
    + 실수형을 정수형으로 형변환할 때 반올림 x

### 5.1 자동 형변환
- 서로 다른 타입간의 뎃셈에서는 두 타입 중 표현범위가 더 넓은 타입으로 형변환하여 타입을 일치시킨 후 연산 수행 (산술 변환)
- byte -> short,char -> int -> long -> float -> double (`->` 방향 자동 형변환)
- short와 char은 서로 형변환 x (값 손실 발생 가능)
- 기본형과 참조형은 서로 형변환 x

# Reference
> 자바의 정석 - 남궁성
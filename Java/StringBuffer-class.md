# StringBuffer클래스

## 1. 특징
- String처럼 문자형 배열을 내부적으로 가지고 있다.
- String클래스와 다르게 내용을 변경할 수 있다.
- 인스턴스를 생성할 때 버퍼의 크기를 충분히 지정해주는 것이 좋다. (버퍼가 작으면 성능이 저하되고 작업 중 더 큰 배열이 추가로 생성된다.)
- 기본 버퍼의 크기는 16이다.
- `주의` : String클래스와 달리 equals()를 오버라이딩하지 않았다 !!

```java
public final class StringBuffer implements java.io.Serializable
{
    private char[] value;
    ...
}
```

```java
StringBuffer sb = new StringBuffer("abc");
sb.append("123");

// append() : 지정된 내용을 StringBuffer에 추가 후 StringBuffer의 참조를 반환한다.
// sb.append("123").append("zz"); 도 가능
```

## 2. StringBuffer클래스의 생성자와 메서드

### 2.1 생성자
- `StringBuffer()`   
: 16문자를 담을 수 있는 버퍼를 가진 StringBuffer 인스턴스를 생성

- `StringBuffer(int length)`   
: 지정된 개수의 문자를 담을 수 있는 버퍼를 가진 StringBuffer인스턴스를 생성

- `StringBuffer(String str)`   
: 지정된 문자열 값을 갖는 StringBuffer인스턴스를 생성 (문자열 길이 + 16)

### 2.2 메서드
- `StringBuffer append(모든 기본형 타입 매개변수(char[], Object, String 포함))`   
: 매개변수로 입력된 값을 문자열로 변환하여 StringBuffer인스턴스가 저장하고 있는 문자열의 뒤에 덧붙인다.

- `int capacity()`   
: StringBuffer인스턴스의 버퍼크기를 알려준다.

- `char charAt(int index)`   
: 지정된 위치에 있는 문자(char)를 반환

- `StringBuffer delete(int start, int end)`   
: 시작 위치부터 끝 위치사이에 있는 문자를 제거 후 남은 문자열 반환 (start <= x < end)

- `StringBuffer deleteCharAt(int index)`   
: 지정된 위치의 문자를 제거 후 남은 문자열 반환

- `StringBuffer insert(int pos, 모든 기본형 타입 매개변수(char[], Object, String 포함))`   
: 두 번째 매개변수로 받은 값을 문자열로 변환하여 지정된 위치에 추가 (pos는 0부터)

- `int length()`   
: StringBuffer인스턴스에 저장되어 있는 문자열의 길이 반환

- `StringBuffer replace(int start, int end, String str)`   
: 지정된 범위의 문자들을 주어진 문자열로 바꾼다.

- `StringBuffer reverse()`   
: StringBuffer인스턴스에 저장되어 있는 문자열의 순서를 거꾸로 나열

- `void setCharAt(int index, char ch)`   
: 지정된 위치의 문자를 주어진 문자로 바꾼다.

- `void setLength(int newLength)`   
: 지정된 크기로 문자열의 길이를 변경 (빈 공간은 널문자로 채운다.)

- `String toString()`   
: StringBuffer인스턴스의 문자열을 String으로 반환

- `String substring(int start) / Stirng substring(int start, int end)`   
: 지정된 범위 내의 문자열을 String으로 뽑아서 반환

## 3. StringBuffer vs StringBuilder
- StringBuffer는 동기화가 되어있지만 StringBuilder는 동기화 x
- 멀티쓰레드가 아닌 싱글 쓰레드의 경우 동기화는 성능을 저하시키기때문에 불필요하다.
- 이럴때 StringBuffer 대신 StringBuilder를 사용한다. (성능 향상 !!)
- StringBuffer를 쓰는 자리에 StringBuilder를 사용하기만 하면 된다. (동기화 기능만 추가된 것일 뿐 나머지 모두 동일 !!)

> 동기화 : 멀티쓰레드에서 데이터 공유를 막는 기능   
> StringBuffer의 성능도 충분히 좋기때문에 성능 향상이 꼭 필요한 경우를 제외하고 기존 코드르 굳이 StringBuilder로 바꿀 필요는 없다.
# Reference
> 자바의 정석 - 남궁성

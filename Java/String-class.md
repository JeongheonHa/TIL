# String 클래스

## 1. 특징
- String인스턴스의 내용은 바꿀 수 없다.
- String class = 데이터(char[]) + 메서드(문자열 관련)

## 2. Object클래스에 정의되어있는 형태
```java
public final class String implements java.io.Serializable, Comparable {
  private char[] value;
  ...
}
```

## 3. String str = "abc" vs String str = new String("abc")
```java
// "abc"를 Constant pool에 저장하여 참조변수들이 공유해서 사용 (주소가 같다)
String str = "abc";
String str2 = "abc";

// "abc"를 갖는 각각의 인스턴스를 생성 (주소가 다름, 거의 사용 x)
String str3 = new String("abc");
String str4 = new String("abc");

// 두 문자열을 비교할 때는 주소가 같기 때문에 equals()를 오버라이딩하여 객체 비교가 아닌 문자열 비교로 만들어서 사용한다 !!
```
## 4. 빈 문자열 ("")
- 내용이 없는 문자열, 크기가 0인 char형 배열을 저장한 문자열
- 문자열을 초기화할 때 자주 사용
```java
// 빈 문자열로 초기화
String s = "";

// 공백으로 초기화 
char c = ' '; // char c = ''; (x)

// 해당 타입의 기본값으로 초기화 하는 것보다 빈 문자열로 초기화하는 것을 선호한다.
```
## 5. String클래스의 생성자와 메서드
### 5.1 생성자
- `String(String s)`   
: 주어진 문자열을 갖는 String인스턴스를 생성   

- `String(char[] value)`   
: 주어진 문자열을 갖는 String인스턴스를 생성   

- `String(StringBuffer buf)`   
: StringBuffer인스턴스가 갖고 있는 문자열과 같은 내용의 String인스턴스르 생성

### 5.2 메서드
- `char charAt(int index)`   
: 지정된 위치에 있는 문자를 반환

- `int compareTo(String str)`   
: 문자열과 사전순서로 비교 (`=` -> 0, `<` -> -1, `>` -> 1)   
   > ###### tip : 해당 문자의 유니코드가 몇번인지 비교해보자 !
- `String concat(String str)`   
: 문자열을 뒤에 덧붙인다.
- `boolean contains(CharSequence s)`   
: 지정된 문자열(s)이 포함되었는지 검사
- `boolean endsWith(String suffix)`   
: 지정된 문자열(suffix)로 끝나는지 검사
- `boolean equals(Object obj)`   
: 매개변수로 받은 문자열(obj)과 String인스턴스의 문자열 비교
- `boolean equalsIgnoreCase(String str)`   
: 문자열과 String인스턴스의 문자열을 대소문자 구분없이 비교
- `int indexOf(int ch)`   
: 주어진 문자가 문자열에 존재하는지 확인하여 위치를 반환 (못찾으면 -1 반환)
- `int indexOf(String str)`   
: 주어진 문자열이 존재하는지 확인하여 그 위치를 반환 (없으면 -1 반환)
- `String intern()`   
: 문자열을 constant pool에 등록 (이미 같은 내용의 문자열이 있을 경우 그 문자열의 주소 값 반환)
- `int lastIndexOf(int ch)`   
: 지정된 문자 또는 문자코드를 문자열의 끝에서부터 찾아 위치를 반환 (못찾으면 -1 반환)
- `int lastIndexOf(String str)`   
: 지정된 문자열을 인스턴스의 문자열 끝에서부터 찾아 위치를 반환 (못찾으면 -1 반환)
- `int length()`   
: 문자열의 길이 반환
- `String replace(char old, char new)`   
: 문자열 중 문자(old)를 새로운 문자(new)로 바꾼 후 문자열을 반환
- `String replace(charSequence old, charSequence new)`   
: 문자열 중 문자열(old)을 새로운 문자열(new)로 바꾼 후 문자열 반환
- `String replaceAll(String regex, String replacement)`   
: 문자열 중 지정된 문자열(regex)과 일치하는 것을 새로운 문자열(replacement)로 모두 변경후 문자열 반환
- `String replaceFirst(String regex, String replacement)`   
: 문자열 중 지정된 문자열과 일치하는 것 중 첫 번째 것만 새로운 문자열로 변경후 문자열 반환 (일치하는 문자열이 더 있을 때 사용하면 유용하다)
- `String[] split(String regex)`   
: 문자열을 지정된 구분자(regex)로 나누어 문자열 배열에 담은 후 반환 (ex)구분자 : , _ ...
- `String[] split(String regex, int limit)`   
: 문자열을 지정된 구분자(regex)로 나누어 문자열 배열에 담은 후 limit 개수 만큼의 문자열을 포함한 배열을 반환 (`주의`: 마지막 구분자 포함 x)
- `boolean startsWith(String prefix)`   
: 주어진 문자열(prefix)로 시작하는지 검사
- `String substring(int begin)` / `String substring(int begin, int end)`    
: 주어진 시작위치부터 끝 위치 범위에 포함된 문자열을 반환 (`주의` : begin <= x < end)
- `String toLowerCase()`   
: String인스턴스에 저장되어있는 모든 문자열을 소문자로 변환하여 반환
- `String toUpperCase()`
: String인스턴스에 저장되어잇는 모든 문자열으 대문자로 변환하여 반환
- `String toString()`   
: String인스턴스에 저장되어있는 문자열 반환
- `String trim()`      
: 문자열의 양쪽 끝의 공백을 제거한 문자열 반환
- `static String valueOf(모든 타입(Object 포함) b)`   
: 지정된 값을 문자열로 변환하여 반환 (참조변수의 경우 toString()을 호출한 결과를 반환)

## 6. join()과 StringJoiner클래스
> join() 과 StringJoiner는 jdk1.8부터 추가
### 6.1 join()
- 여러 문자열 사이에 구분자를 넣어 결합한 문자열을 반환
```java
String animals = "dog, cat, bear";
String[] arr = animals.split(",");
String str = String.join("_", arr);
System.out.println(str);  // dog-cat-bear
```

### 6.2 java.util.StringJoiner
- 구분자를 넣어 결합하는 또 다른 방법
```java
StringJoiner sj = new StringJoiner("," , "[" , "]");
String[] strArr = { "aaa", "bbb", "ccc" };

for(String s : strArr)
  sj.add(s.toUpperCase());
  
System.out.println(sj.toString());  // [AAA,BBB,CCC]
```
## 7. 문자열과 기본형간의 변환
- 기본형 값 -> 문자열
```java
int i = 100;
// 첫번째 방법
String str1 = i + "";             // 가독성이 좋다
// 두번째 방법
String str2 = String.valueOf(i);  // 성능이 좋다 (속도가 빠르다)

// 가독성을 1순위로 두고 코딩을 해야한다. (성능은 2순위)
```

- 문자열 -> 기본형 값
```java
// 첫번째 방법
int i = Integer.parseInt("100");
// 두번째 방법
int i2 = Integer.valueOf("100");  // jdk1.5 이후
// 세번째 방법
char c = "A".charAt(0); // "A" -> 'A'

// 두번째 방법은 첫번째 방법에서 parse가 붙은 메서드들은 각각의 타입에서 문자열로 변환한다는 기능은 같지만 
// 이름이 다르기때문에 jdk1.5 이후부터 valueOf()메서드를 추가했다.
// 반환타입이 int가 아닌 Integer이지만 Autoboxing기능으로 Interger를 int로 바꿔주기 때문에 
// int i = Integer.valueOf()를 써도 상관없다.
```


# Reference
> 자바의 정석 - 남궁성

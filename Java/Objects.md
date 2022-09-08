# Objects class

## 1. Objects 클래스
- Object클래스의 보조 클래스
- 모든 메서드가 `static`
- 객체의 비교나 null체크에 유용
- `java.util`패키지

### 1.1 isNull, nonNull()
- 해당 객체가 null인지 확인
```java
static boolean isNull(Object obj)   // 해당 객체가 null이면 true, 아니면 false
static boolean nonNull(Object obj)  // 해당 객체가 null이면 false, 아니면 true
```

### 1.2 requireNonNull()
- 해당 객체가 null이 아니어야 하는 경우에 사용 
- 해당 객체가 null이면 NullPointerException예외 발생
- 매개변수 객체의 유효성 검사에 유용하다.
```java
static <T> T requireNonNull(T obj)
static <T> T requireNonNull(T obj, String message)
static <T> T requireNonNull(T obj, Supplier<String> messageSupplier)

// 예제
// requireNonNull()사용 x
void setName(String name) {
    if(name == null)
        throw new NullPointerException("name must not be null");
    this.name = name;
}
// requireNonNull()사용
void setName(String name) {
    this.name = Objects.requireNonNull(name, "name must not be null");
}
```

### 1.3 compare()
- 대소 비교에 유용 (equals는 등가 비교)
```java
static int compare(Object a, Object b, Comparator c)
```

### 1.4 equals() & deepEquals()
- `equals()` : 내부에서 두 객체가 null인지 검사한다는 점을 제외하곤 Object클래스의 equals와 같다.
- `deepEquals()` : 다 차원 배열의 두 객체 비교
```java
static boolean equals(Object a, Object b) {
    return (a == b) || (a != null && a.equals(b));
}
static boolean deepEquals(Object a, Object b)
```

### 1.5 toString()
- `toString(Object o)` : 내부에서 객체가 null인지 검사한다는 점을 제외하곤 Object클래스의 toString과 같다.
```java
static String toString(Object o)    // 객체의 정보를 문자열로 반환 (내부에서 객체가 null인지 검사)
static String toString(Object o, String nullDefault)    // 지정된 객체가 null일 경우 대신 사용할 값을 지정
```

### 1.6 hashCode()
- `hashCode(Object o)` : 내부에서 객체가 null인지 검사한다는 점을 제외하곤 Object클래스의 hashCode와 같다.
```java
static int hashCode(Object o)   // 지정된 객체의 hashCode를 반환 (내부에서 객체가 null인지 검사, null일 경우 0 반환)
static int hash(Object... values)   // 가변인자를 이용해 hashCode 반환
```

# Reference
> 자바의 정석 - 남궁성

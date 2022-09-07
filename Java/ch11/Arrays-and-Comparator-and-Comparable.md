# Arrays & Comparator & Comparable

## 1. Arrays
- 배열을 다루기 편리한 메서드 제공 (모두 `static` 메서드)

### 1.1 1차원 배열의 비교와 출력
- `equals()` : 두 배열에 저장된 모든 요소를 비교 (true/false)
- `toString()` : 배열의 모든 요소를 문자열로 출력 (1차원만 가능)

```java
// equals(), toString()
String[] arr = {"aaa", "bbb"};
String[] arr2 = {"aaa", "bbb"};
System.out.println(Arrays.equals(arr, arr2))    // true
System.out.println(Arrays.toString(arr))    // [aaa, bbb]

// toString() 정의
static String toString(boolean[] a)
static String toString(byte[] a)
static String toString(char[] a)
static String toString(short[] a)
static String toString(int[] a)
static String toString(long[] a)
static String toString(float[] a)
static String toString(double[] a)
static String toString(Object[] a)
```

### 1.2 다차원 배열의 비교와 출력
- `deepEquals()` : 다차원 배열을 비교
- `deepToString()` : 다차원 배열의 모든 요소를 문자열로 출력 (2D,3D... 가능)

```java
// deepEquals()
String[][] str2D = new String[][] {{"aaa", "bbb"}, {"AAA", "BBB"}};
String[][] str2D2 = new String[][] {{"aaa", "bbb"}, {"AAA", "BBB"}}
System.out.println(Arrays.deepEquals(str2D, str2D2));   // true

// deepToString()
int[][] arr2D = {{11,12}, {21,22}};
System.ou.println(Arrays.deepToString(arr2D));
```


### 1.3 배열의 복사
- `copyOf()` : 배열 전체를 복사해서 새로운 배열을 만들어 반환
- `copyOfRange()` : 배열의 일부를 복사해서 새로운 배열을 만들어 반환 (마지막 미포함)

```java
// copyOf()
int[] arr = {0,1,2,3,4};
int[] arr2 = Arrays.copyOf(arr, arr.length);    // [0,1,2,3,4]
int[] arr3 = Arrays.copyOf(arr, 3);             // [0,1,2]
int[] arr4 = Arrays.copyOf(arr, 7);             // [0,1,2,3,4,0,0]
// copyOfRange()
int[] arr5 = Arrays.copyOfRange(arr,2,4);       // [2,3]
int[] arr6 = Arrays.copyOfRange(arr,0,7);       // [0,1,2,3,4,0,0]

```

### 1.4 배열 채우기
- `fill()` : 배열의 모든 요소를 지정된 값으로 채운다.
- `setAll()` : 배열을 채우는데 사용할 `함수형 인터페이스`를 `매개변수`로 받는다.
    + 이 메서드를 호출할 때는 `함수형 인터페이스를 구현한 객체`를 매개변수로 지정하던가 `람다식`을 지정해야한다.

```java
int[] arr = new int[5];
Arrays.fill(arr,9); // [9,9,9,9,9];
Arrays.setAll(arr, () -> (int)(Math.random() * 5) + 1); // [1,5,2,1,1]
```

### 1.5 배열의 정렬과 검색
- `sort()` : 배열을 `정렬`한다.
- `binarySearch()` : 배열에서 `지정된 값`이 저장된 `위치`를 찾아서 `반환` (이진 탐색)
    + 반드시 정렬된 상태에서 사용해야 올바른 값을 얻을 수 있다.
    + 검색한 값과 일치하는 요소가 중복되게 있는 경우 어떤 것의 위치가 반환될지 알 수 없다.
- 순차 탐색 vs 이진 탐색
    + `순차 탐색` : 배열의 첫 번째 요소부터 순서대로 하나씩 비교 (검색속도 `느리다`)
    + `이진 탐색` : 배열 범위를 반절씩 줄여가며 비교 (검색속도 `빠르다`)

```java
int[] arr = {3,2,4,1,0};
Arrays.sort(arr);
System.out.println(Arrays.toString(arr));   // [0,1,2,3,4]
int idx = Arrays.binarySearch(arr,2);       // 2
```

### 1.6 배열을 List로 변환
- `asList(Object... a)` : 배열을 List에 담아서 반환
    + 매개변수 타입이 `가변인수`라서 배열 생성없이 저장할 요소들만 나열하는 것도 가능
    + `주의` : asList()가 반환한 List는 읽기 전용이기 때문에 List의 크기는 변경할 수 없다. (`추가/삭제` x, 저장된 내용 `변경` o)

```java
// asList(Object... a)
// <1> 배열을 생성한 경우
List list = Arrays.asList(new Integer[]{1,2,3,4,5});
// <2> 배열을 생성하지 않는 경우
List list = Arrays.asList(1,2,3,4,5);
// list.add(6); // UnsupprotedOperationException 예외 발생

// asList()가 반환한 List의 크기를 변경하고 싶을 경우
List list = new ArrayList(Arrays.asList(1,2,3,4,5));    // 새로운 ArrayList 객체 같은 내용으로 만든다.
```

## 2. Comparator & Comparable
- 객체를 정렬하는데 필요한 메서드를 정의한 인터페이스 (정렬 기준을 제공)
    + `Comparable` : `기본 정렬기준`을 구현하는데 사용하는 인터페이스 (`java.lang`패키지에 있어서 `import 필요 x`)
    + `Comparator` : 기본 정렬기준 외에 `다른 기준`으로 정렬하고자할 때 사용하는 인터페이스 (`java.util`패키지에 있기 때문에 `import 필요`)

```java
// 정의
public interface Comparable {
    public int compareTo(Object o); // 주어진 객체(o)를 자신(this)와 비교 (음수,0,양수)
}

public interface Comparator {
    int compare(Object o1, Object o2);  // o1, o2 두 객체 비교 (왼쪽이 작다 : 음수, 같다 : 0, 왼쪽이 크다 : 양수)
    // Comparator를 구현하는 클래스는 equals를 오버라이딩해야할 수 도 있다는 것을 알리기위해 정의
    boolean equals(Object obj);
}
```

### 2.1 sort()
- 두 대상을 `비교`해서 `자리 바꿈`을 반복하는 것
- 정렬을 하기위해서는 `정렬 대상`(a)과 `정렬 기준`(c)이 필요하다.

```java
// 정의

// 객체 배열에 저장된 객체가 구현한 Comparable(정렬 기준)에 의한 정렬
static void sort(Object[] a)
// 지정된 Comparator에 의한 정렬
static void sort(Object[] a, Comparator c)
```
- Integer & Comparable
```java
// 정렬 대상과 정렬 기준을 매개변수로 받는다.
static void sort(int[] intArr) {    // int타입이 Comparable을 구현하기 때문에 정렬 기준 없이 대상만 매개변수로 받는다.
    for(int i = 0; i < intArr.length-1;i++) {
        for(int j = 0; j < intArr.length-1-i; j++) {
            int tmp = 0;
            // 정렬 기준을 정한다.
            if(int Arr[j] > intArr[j+1]) {  // 이 부분만 바뀔 뿐 전체적인 정렬 방식은 변하지 않는다.
                // 자리 바꿈
                tmp = int intArr[j];
                intArr[j] = intArr[j+1];
                intArr[j+1] = tmp;
            }
        }   // for
    }   // for
}
```

### 2.2 compareTo()
```java
public final class Integer extends Number implements Comparable {
    ...
    public int compareTo(Integer anotherInteger) {
        // ... 에 선언되었을 Integer클래스 자신의 value값
        int thisVal = this.value;
        // 매개변수로 받은 value값
        int anotherVal = anotherInteger.value;
        // 두 개를 삼항 연산자를 통해 비교
        return (thisVal < anotherVal ? -1 : (thisVal == anotherVal ? 0 : 1));
    }
}
```
> `주의` : 음수나 양수이면 되기 때문에  꼭 -1,1이 아니더라도 뺄셈(-)을 이용하여 두 객체의 차이가 -231312이가나 231231도 비교가 가능하다.   
> 하지만, compareTo()메서드는 int형을 반환하기 때문에 값이 int형의 최대값을 초과한다면 overflow와 underflow의 위험성이 있어   
> 비교 연산자를 사용하는 것이 안전하다.

### 2.3 정렬 직접 구현
```java
// 내림차순
class Descending implements Comparator {
    public int compare(Object o1, Object o2) {
        // 객체가 Comparable인터페이스로 형변환이 가능한지 확인
        if(o1 instanceof Comparable && o2 instanceof Comparable) {
            // Comparable인터페이스로 형변환
            Comparable c1 = (Comparable)o1;
            Comparable c2 = (Comparable)o2;
            // 매개변수로 받은 객체(c1)에 구현된 compareTo()를 사용하여 c1의 객체(자신)와 c2의 객체를 비교
            return c1.compareTo(c2) * -1;
        }
        return -1;
    }
}
```
# Reference
> 자바의 정석 -남궁성

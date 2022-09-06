# array

## 1. 배열
- 같은 타입의 여러 변수를 하나의 묶음으로 다루는 것
- index의 범위는 0부터 '배열길이 -1'까지
- index로 상수 대신 변수나 수식도 가능
- 배열의 길이는 양의 정수이어야 하고 최대값은 int타입의 최대값(약 20억)이다. (0포함)
- 배열의 길이는 한번 선언되면 변경할 수 없다. (복사를 이용해야한다.)

### 1.1 배열의 초기화
```java
int[] score = new int[5];
int[] score = {50, 60, 70, 80}; // new int[] 생략

// 선언과 초기화를 따로 하는 경우
int[] score;
score = {50, 60, 70, 80};

// 길이가 0인 배열
int[] score = {};

// 배열 출력
import java.util.*;
System.out.println(Arrays.toString(배열 이름)); // [첫 번째 요소, 두 번째 요소 ... ]
// 그냥 배열 이름만 출력하는 경우
System.out.println(배열 이름);  // 타입@주소
// char배열 출력
char[] chArr = {'a', 'b', 'c', 'd'};
System.out.println(chArr);  // abcd
```

### 1.2 배열의 복사
- `System.arraycopy()` 이용
- 배열의 위치가 적절하지 못한 경우 `ArrayIndexOutOfBoundsException`발생
```java
System.out.println(num, 0, newNum, 0, num.length);  // num[0]에서 newNum[0]으로 num.length개의 데이터 복사
```

### 1.3 값 바꾸기
```java
int[] arr = {0, 1, 2, 3, 4, 5};

int tmp = arr[0];
arr[0] = arr[1];
arr[1] = tmp;
```

## 2. String 배열

### 2.1 String 배열의 선언과 생성
```java
String name = new String[3];
```

- 타입에 따른 변수의 기본값

| 자료형 | 기본값 |
|---|---|
| boolean | false |
| char | '\u0000' |
| byte, short, int | 0 |
| long | 0L |
| float | 0.0f |
| double | 0.0d or 0.0 |
| 참조형 변수 | null |

### 2.2 String 배열의 초기화
- 참조형 배열의 경ㅇ 배열에 저장되는 것은 객체의 주소
```java
String[] name = new String[] {"Kim", "Park", "Yi"};
String[] name = {"Kim", "Park", "Yi"};
String[] name = new String[3];
name[0] = "Kim";    // new String("Kim");과 같다.
name[1] = "Park";
name[2] = "Yi";
```

### 2.3 char배열과 String클래스
- String클래스는 char배열에 기능(메서드)을 추가한 것이다.
- char배열과 String클래스의 차이는 String객체는 읽을 수 만있을 뿐 내용을 변경할 수 없다.
- 기본형 변수의 값을 비교할 경우 '=='연산자를 사용하지만, 문자열의 내용을 비교할 때는 equals()를 사용한다.
- char배열 -> String클래스 변환
```java
char[] chArr = {'A', 'B', 'C'};
String str = new String(chArr);
char[] tmp = str.toCharArray();
```

### 2.4 커맨드 라인을 통해 입력받기
```java
> java 클래스이름 [0] [1] [2] ...
```

## 3. 다차원 배열

### 3.1 2차원 배열의 선언과 인덱스
```java
타입[][] 변수이름;
```
```java
// 2차원 배열 생성
int[][] score = new int[4][3];
```

### 3.2 2차원 배열의 초기화
```java
int[][] arr = new int[][] { {1, 2, 3}, {4, 5, 6} };
int[][] arr = { {1, 2, 3}, {4, 5, 6} };
```

### 3.3 2차원 배열의 향상된 for문
```java
int[][] score = {
                      {100, 100, 100}
                    , {20, 20, 20}
                    , {30, 30, 30}
                    , {40, 40, 40}
};

int sum = 0;

for(int[] tmp : score) {
    for(int i : tmp) {
        sum += i;
    }
}
```

### 3.4 가변 배열
```java
// <1>
int[][] score = new int[3][];
score[0] = new int[4];
score[1] = new int[3];
score[2] = new int[2];

// <2>
int[][] score = {
                      {100, 100, 100, 100}
                    , {20, 20, 20}
                    , {30, 30}
};
```

# Reference
> 자바의 정석 - 남궁성

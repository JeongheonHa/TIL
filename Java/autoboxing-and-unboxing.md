# autoboxing & unboxing

## 1. 설명
- autoboxing : `기본형`의 값을 `객체`로 자동변환하는 것
- unboxing : `객체`를 `기본형`의 값으로 자동변환하는 것
- jdk1.5 부터 추가되었다.
> jdk1.5이전에는 기본형과 참조형간의 연산의 불가능했기 때문에 래퍼 클래스로 기본형을 객체로 만들어서 연산해야 했다.
```java
// 기본형
int i = 5;
// 참조형
Integer iObj = new Integer(7);

// 기본형과 참조형간의 연산
int sum = i + iObj; // 사실은, int sum = i + iObj.intValue();로 컴파일러가 자동으로 변환해주는 것이다.
```

## 2. 예제
```java
int i = 10;

// 기본형을 참조형으로 형변환 (생략도 가능)
Integer intg = (Integer)i;  // 컴파일 후 : Integer intg = Integer.valueOf(i);
Object obj = (Object)i;     // 컴파일 후 :Object obj = (Object)Integer.valueOf(i);

Long lng = 100L;            // 컴파일 후 : Long lng = new Long(100L);

// 참조형을 기본형으로 형변환 (생략도 가능)
Integer intg2 = new Ingeter(20);
int i3 = (int)intg2;
```

# Reference
> 자바의 정석 - 남궁성

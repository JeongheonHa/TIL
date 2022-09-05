# 조건문 & 반복문

## 1. 조건문
- if문
```java
if (조건식1) {
    // 조건이 true면 실행할 문장
} else if (조건식2) {
    // 조건이 true면 실행할 문장
} else if (조건식3) {
    // 조건이 true면 실행할 문장
} else {    // else 블럭은 생략 가능
    // 어떤 조건도 해당하지 않을 때 실행할 문장
}
```

- 중접 if문
```java
if (조건식1) {
    if (조건식2) {
        // 조건1,2가 모두 참일 때 실행할 문장
    } else {
        // 조건1만 참일 때 실행할 문장
    }
} else {
    // 조건1이 false일 때 실행할 문장
}
```

- switch문
    + switch문의 조건식 결과는 `정수` or `문자열`이어야 한다.
    + case문의 값은 `정수`,` 상수`, `문자열 리터럴`만 가능하며, 중복되지 않아야 한다.
```java
class Ex {
    public static void main(String[] args) {
        final int MONTH = 11;

        switch(MONTH) {
            case 3:
            case 4:
            case 5:
                System.out.println("봄");
            case 6: case 7: case 8:
                System.out.println("여름");
            case 9: case 10: case 11:
                System.out.println("가을");
            default:
                System.out.println("겨울");
        }
    }
}
```

## 2. 반복문
- for문 
    + for문 내의 불필요한 변수의 사용을 줄이는 것이 좋다.
```java
for(초기화; 조건식; 증감식) {
    // 수행될 문장
}

// 예제
for(int i = 1, j = 10; i <= 10; i++, j--) { // 가능
    // 수행될 문장
}
```
- 향상된 for문
    + 배열이나 컬렉션에 저장된 요소를 읽어오는 용도로만 사용 가능
```java
for(타입 변수명 : 배열 or 컬렉션) {
    // 반복할 문장
}
```
- while문
    + 조건식 생략 불가
    + 조건식에 0이 아닌 양수를 넣을 경우 무한 루프
```java
while(조건식) {
    // 조건식이 참일 경우 수행될 문장
}
```

- do-while문
    + 최소 한번은 수행될 것을 보장
```java
do {
    // 조건식의 연산결과가 참일 때 수행될 문장
} while(조건식);
```

- `break`   
: 자신이 포함된 가장 가까운 루프 탈출
- `continue`    
: 반복문의 끝으로 이동하여 다음 반복으로 넘어간다.

- 이름 붙은 반복문
```java
class Ex {
    public static void main(String[] args) {
        Loop : for(int i = 2; i <= 9; i++) {
            for(int j = 1; j <= 9; j++) {
                if(j==5)
                    break Loop;
                System.out.println(i + "*" + j + "=" + i*j);
            }
            System.out.println();
        }
    }
}
```

# Reference
> 자바의 정석 - 남궁성

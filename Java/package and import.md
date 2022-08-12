# 3. 패키지
## 3.1 패키지
- 서로 관련된 클래스(.class)와 인터페이스의 묶음
- 간단히 패키지가 폴더라면 클래스는 파일
- 패키지가 선언되지 않은 클래스는 자동적으로 default 패키지에 속한다.
- bin : .class파일 위치
- src : source파일 위치

## 3.2 클래스 패스
- 클래스파일을 찾는 경로
- ;로 구분
- 환경변수에 추가

## 3.4 import
- 사용할 클래스가 속한 패키지르 지정
- 해당 패키지 안에 있는 class 사용시 패키지명 생략
- java.lang 기본적인 패키지로 생략 가능
- class이름 대신 * 사용시 패키지의 모든 class를 import한다는 뜻 (패키지 이름 대신은 안됨)
- 이름이 같은 class를 import할 경우 패키지명을 붙인다.

## 3.5 static import
- static 멤버 사용시 클래스 이름 생략
```java
import static java.lang.Integer.*;    // Integer class의 모든 static 멤버
import static java.lang.Math.random;  // Math.random() 만
import static java.System.out;        // System.out -> out 만으로 참조 가능

// System.out.println(Math.random());
out.println(random());
```
# Reference
> 자바의 정석 - 남궁성

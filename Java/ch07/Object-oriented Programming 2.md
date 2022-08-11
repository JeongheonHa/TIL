# 1. 상속
### 예제 : [github 링크](https://github.com/JeongheonHa/Java-Archive/tree/main/Java/src)
## 1.1 상속
- 두 클래스를 조상과 자손관계로 맺어주는 것.
- 자손 클래스는 모든 멤버(변수, method)를 상속 받는다. (생성자, 초기화 블록은 제외)
- 자손의 멤버 수는 조상보다 항상 크거가 같다.
- 자손 클래스의 인스턴스 생성시 조상 클래스의 멤버도 같이 생성.
- Java는 단일 상속만을 허용 (같은 이름의 method가 있을 경우 충돌이 발생할 우려가 있기 때문에)
```java
class Circle extends Point {
  int r;
}
```
## 1.2 포함
- 한 클래스의 멤버 변수로 다른 클래스의 참조 변수를 선언하는 것.   
```java
class Circle {
  Point c = new Point();
  int r;
}
```
## 1.3 상속 vs 포함
- Circle **is a** Point (상속)
- Circle **has a** Point (포함)

## 1.4 Object class
- 모든 클래스 상속계층도의 최상위에 있는 조상 클래스.
- 부모가 없는 클래스는 모두 Object를 상속하고 있다.

# 2. 오버라이딩
## 2.1 오버라이딩 (덮어쓰기)
- 조상 클래스로부터 상속받은 메서드의 내용을 변경하는 것 (header는 그대로, **body**의 내용만 바꾼다)
- 선언부 (이름, 반환 타입, 매개 변수 목록) 일치 // jdk1.5부터 반환 타입 완화
- 주의 사항
  * 접근 제어자를 조상 클래스의 메서드보다 좁은 범위로 변경할 수 없다.
  * 예외는 조상 클래스의 메서도보다 많이 선언할 수 없다.
  * 인스턴스 메서드를 static 메서드로 또느 그 반대로 변경할 수 없다.

## 2.2 오버로딩 vs 오버라이딩
- 오버로딩 : 한 클래스 내에 이름만 같은 여러 메서드를 생성 (차이 : 이름 o, 매개변수 개수 or 매개변수 타입 x, 반환타입은 영향 x)
- 오버라이딩 : 상속 받은 메서드의 내용을 변경 (차이 : 이름 o, 매개변수 o, 반환 타입 o)

## 2.3 super
- 조상멤버, 자식멤버를 구별하는데 사용되는 참조 변수. (this는 지역변수와 인스턴스 변수 구별하는 참조 변수)
- this와 마찬가지로 static method에서 사용 불가

## 2.4 super()
- 조상 클래스의 생성자
- 자손 클래스의 인스턴스를 생성하면, 자손의 멤버와 조상의 멤버가 함쳐진 하나의 인스턴스가 생성
- 생성된 인스턴스에 조상 멤버를 초기화 시켜주기위해 조상의 생성자 호출
- Object클래스르 제외한 모든 클래스의 생성자 첫줄에는 this() or super() 호출 (없으면 컴파일러가 자동으로 super(); 호출)   

**기본 생성자**는 필수!!!!!!!!
------------------   

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
# 4. 제어자
## 4.1 제어자
- 접근 제어자 : public, protected, default, private (4개 중 하나만 사용가능)
- 그 외 : static, final, abstract, native, transient, synchronized, volatile, strictfp
- 순서 : 접근 제어자 -> 그 외

## 4.2 static
- 대상
  * 멤버변수 - static 변수가 된다.
  * 메서드 - static method가 된다.

## 4.3 final
- 대상
  * 클래스 : 자손이 없는 클래스가 된다.
  * 메서드 : 오버라이딩 x
  * 멤버변수 + 지역변수 : 값 변경 x (상수), 인스턴스 변수의 경우 생성자에서 초기화

## 4.4 abstract
- 대상
  * 클래스 : 추상메서드가 있으면 추상 클래스
  * 메서드 : header만 있고 body는 없는 메서드

## 접근 제어자
제어자 | 같은 클래스 | 같은 패키지 | 자손 클래스 | 전체 |
---|---|---|---|---|
public| o | o | o | o |
protected| o | o | o |
(default)| o | o |
private| o |

# 5. 다형성
## 5.1 다형성
- 조상티입의 참조변수로 자손타입의 객체를 다룰 수 있는 것 (반대로 자손타입 참조변수 -> 조상타입 객체 x)

## 5.2 참조변수의 형변환
- 사용할 수 있는 멤버의 개수를 조절하는 것 (주소값, 객체 안바뀜)
- 생략 또한 가능 (참조할 수 있는 개수범위 내에 있으며 가능, 개수 초과면 x)
- *객체는 그대로있고 형변환만 한다는 거 주의!!!*

## 5.3 instanceof 연산자
- 참조변수의 형변환 가능여부를 확인할 때 사용
- 가능하면 True 반환 (조상 또한 true가 반환된다는 점 주의하자)
- *형변환 전에 **반드시** instanceof로 확인*



-------------
# Reference
>자바의 정석 - 남궁성





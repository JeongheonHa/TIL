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

-------------
# Reference
>자바의 정석 - 남궁성





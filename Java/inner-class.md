 # 내부 클래스

## 1. 내부 클래스
- 클래스 안에 선언된 클래스
- 내부 클래스에서 외부 클래스 멤버들 쉽게 접근 가능 (객체 생성없이 접근)
- 내부 클래스가 외부 클래스의 멤버로서 간주되며 인스턴스 멤버와 static멤버간의 규칙이 똑같이 적용된다.
- 캡슐화

## 2. 종류
> 변수의 성질과 범위가 유사하므로 비교하면서 이해해보자    
> 선언 위치는 변수와 동일
```java
class Outer {
  class InstanceInner {}      // 인스턴스 클래스 : 인스턴스 멤버처럼 사용
  static class StaticInner {} // 스태틱 클래스 : 스태틱 멤버처럼 사용
  
  void myMethod() { 
    class LocalInner {}       // 지역 클래스 : 메서드 or 초기화 블럭에 선언, 선언된 영역 내에서만 사용
  }
}

// 익명 클래스 : 클래스의 선언과 객체의 생성을 동시에 하는 이름없는 클래스(일회용)
```

## 3. 제어자
- 내부 클래스의 접근제어자는 변수와 동일
- static 클래스만 static멤버를 정의할 수 있다. (즉, 나머지 인스턴스 내부 클래스와 지역 내부 클래스는 static 변수를 정의할 수 없다)
- final static 접근자를 가진 변수는 상수이므로 모든 내부 클래스에서 정의가 가능하다.
- 지역 내부 클래스의 final static 상수는 메서드 내에서만 접근할 수 있다는 점을 주의하자 !
- 외부 클래스의 지역변수는 final이 붙은 상수만 접근 가능 (지역 클래스의 인스턴스가 소멸된 지역 변수를 참조할 수 있기 때문에)
- 지역변수 접근 : 변수이름 / 내부 클래스 변수 접근 : this.변수이름 / 외부 클래스 변수 접근 : 외부 클래스 이름.this.변수이름
```java
class Inner {	// 외부 클래스
  class InstanceInner { // instance 내부 클래스
    int iv = 100;
    static int cv = 100;  // error
    final static int CONST = 100; // 상수
  }
  
  static class StaticInner {	// static 내부 클래스
  int iv = 200;
  static int cv = 200;	// static 멤버
  }
  
  void myMethod() {
    class LocalInner {  // Local 내부 클래스
      int iv = 300;
      // static int cv = 300; // error
      final static int CONST = 300; // 상수
    }
}
```
```java
class Inner {
  class InstanceInner {}
  static class StaticInner {}
  // static StaticInner cv = new instanceInner();	 // error

  static void staticMethod() {
    // InstanceInner obj1 = new instanceInner();	 // error
  }
}
```
```java
/* 객체 생성하는 방법 */
Outer oc = new Outer(); // 외부 클래스 객체 생성
Outer.InstanceInner ii = oc.new InstanceInner();  // 인스턴스 내부 클래스 객체 생성 (외부 클래스 객체 먼저 생성 후 내부 클래스 생성)
Outer.StaticInner.cv; // 객체 생성안하고 사용 가능 (static 내부 클래스)
Outer.StaticInner si = new Outer.StaticInner(); // 바로 스태틱 내부 클래스 객체 생성 (스태틱 내부 클래스의 인스턴스는 외부 클래스를 먼저 생성하지 않아도 된다.)
```
## 4. 익명 클래스
- 익명 클래스 : 클래스의 선언과 객체의 생성을 동시에 하는 이름없는 클래스(일회용)
```java
class Inner {
  public static void main(String[] args) {
    Button b = new Button("Start");
    b.addActionListener(new ActionListener() {
        public void actionPerformed(ActionEvent e) {
          System.out.println("ActionEvent occurred!!");
        }
      }
    };  // 익명 클래스
  }  // main
} // Inner class
```
# Reference
> 자바의 정석 - 남궁성

# Object클래스

## 1. Object클래스
- 모든 클래스의 `조상`클래스

## 2. Object클래스의 메서드
| 메서드 | 설명 |
|---|---|
| protected Object clone() | 객체 자신의 `복사본`을 `반환` |
| public boolean equals(Object obj) | 객체 자신과 객체 obj가 같은 객체인지 알려준다. (true/false) |
| protected void finalize() | 객체가 소멸될 때 가비지 컬렉터에 의해 자동적으로 호출 </br> 이 때 수행되어야하는 코드가 있을 때 오버라이딩한다. (`거의 사용 x`) |
| public Class getClass() | 객체 자신의 `클래스 정보`를 담고 있는 Class인스턴스를 `반환` |
| public int hashCode() | 객체 자신의 `해시코드`를 `반환` |
| public String toString() | 객체 자신의 정보를 `문자열`로 `반환` |
| public void notify() | 객체 자신을 사용하려고 기다리는 쓰레드를 하나만 깨운다. |
| public void notifyAll() | 객체 자신을 사용하려고 기다리는 모든 쓰레드를 깨운다. |
| public void wait() </br> public void wait(long timeout) </br> public void wait(long timeout, int nanos) | 다른 쓰레드가 notify()나 notifyAll()을 호출할 때까지 </br> 현재 쓰레드를 무한히 또는 지정된 시간(timeout, nanos)동안 기다리게 한다. |
> timeout : 천 분의 1초, nanos : 10^9분의 1초

## 3. equals(Object obj) 
- 객체의 주소를 비교하는 것이기 때문에 값을 비교하도록 `오버라이딩`해서 사용하도록 하자 
```java
public boolean equals(Object obj) {
    return (this == obj)
}
```

## 4. hashCode()
- 해싱기법에 사용되는 해시함수를 구현한 것이다.
- 해시함수는 찾고자하는 값을 입력하면, 그 값이 저장된 위치를 알려주는 해시코드를 반환한다.
- Object클래스에 정의된 hashCode메서드는 객체의 주소값으로 해시코드를 만들어 반환하기 때문에 `32bit JVM`에서는 서로 다른 두 객체는 결코 같은 해시코드를 가질 수 없지만, `64bit JVM`에서는 8byte 주소값으로 해시코드(4byte)를 만들기 대문에 해시코드가 중복될 수 있다.
- `hashCode`메서드도 적절히 `오버리이딩`해서 사용한다.

## 5. toString()
- 인스턴스에 대한 정보를 문자열로 제공할 목적으로 정의한 것이다.
- Object클래스의 toString()을 호출하면 클래스이름에 16진수의 해시코드를 얻게 되므로 문자열로 `오버라이딩`해서 사용하도록 하자.
- `String클래스`에는 toString()은 String인스턴스가 갖고 있는 문자열을 반환하도록 `오버라이딩` 되어있다.

```java
public String toString() {
    return getClass().getName()+"@"+Integer.toHexString(hashCode());
}
```

## 6. clone()
- Object클래스에 정의된 clone()은 단순히 인스턴스변수의 값만 복사하기 때문에    
참조타입의 인스턴스 변수가 있는 클래스는 완전한 인스턴스 복제가 이루어지지 않는다.
- clone()을 사용 방법
    + 먼저 복제할 클래스가 `Cloneable인터페이스`를 `구현`해야하고
    + clone()을 오버라이딩하면서 접근 제어자를 `protected`에서 `public`으로 변경한다.
    + 마지막으로 조상클래스의 clone()을 호출하는 코드가 포함된 `try-catch문`을 `작성`한다.

```java
public class Object {
    ...
    protected native Object clone() throws CloneNotSupportedException;
    ...
}
```
```java
class Point implements Cloneable {
    ...
    public Object clone() {
        Object obj = null;
        try {
            obj = super.clone();
        } catch(CloneNotSupportedException e) {}
        return obj;
    }
}
```

### 6.1 공변 반환타입
- 오버라이딩할 때 `조상 메서드의 반환타입`을 `자손 클래스의 타입`으로 `변경`을 허용하는 것
```java
// 반환타입을 Object -> Point 변경
public Point clone() {
    Object obj = null;
        try {
            obj = super.clone();
        } catch(CloneNotSupportedException e) {}
        // Pointㅏ입으로 형변환
        return (Point)obj;
}
```

### 6.2 얕은 복사, 깊은 복사
- `얕은 복사`
    + 단순히 객체에 저장된 값을 그대로 복제할 뿐, 객체가 참조하고 있는 객체까지 복제하지 않는다.
    + 원본과 복사본이 같은 객체를 공유하므로 원본을 변경하면 복사본도 영향을 받는다.
- `깊은 복사`
    + 참조하고 있는 객체까지 복제하는 것
    + 원본과 복사본이 서로 다른 객체를 참조하기 때문에 원본의 변경이 복사본에 영향을 미치지 않는다.

## 7. getClass()
- 자신이 속한 클래스의 `Class객체`를 반환하는 메서드
- Class객체는 클래스의 모든 정보를 담고 있으며, `클래스 당 1개`만 존재한다.
- Class객체는 클래스 파일이 `클래스로더`에 의해서 메모리에 올라갈 때 자동으로 생성된다.

### 7.1 Class객체 얻는 방법
```java
// 생성된 객체(Card)로부터 얻는 방법
Class cObj = new Card().getClass();
// 클래스 리터럴(*.class)로부터 얻는 방법
Class cObj = Card.class;
// 클래스 이름으로부터 얻는 방법
Class cObj = Class.forName("Card");
```

# Reference
> 자바의 정석 - 남궁성

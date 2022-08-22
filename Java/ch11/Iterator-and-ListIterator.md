# Iterator & ListIterator

## 1. Enumeration, Iterator, ListIterator
- 컬렉션에 저장된 데이터를 접근하는데 사용되는 인터페이스
- Enumeration : Iterator의 구버전
- ListIterator : Iterator의 기능을 향상시킨 것 (`단뱡향` -> `양방향`)

## 2. Iterator
- 컬렉션에 저장된 요소들을 읽어오는 방법을 `표준화`한 것
- 컬렉션에 `iterator()`를 `호출`해서 Iterator를 구현한 객체를 얻어서 사용
- Iterator는 `1회용`이기 때문에 다쓰고나면 다시 얻어와야 한다.

```java
// 정의
public interface Iterator {
    boolean hasNext();
    Object next();
    void remove();
}
public interface Collection {   // Collection은 List와 Set의 조상
    ...
    public Iterator iterator();
    ...
}
```
```java
// Iterator 사용 예

// 다형성을 이용하여 List객체 생성
Collection c = new ArrayList(); // 다형성을 이용할 경우 ArrayList()부분만 바꿔주면 되기때문에 편리하다.
// Iterator를 구현한 객체를 반환하여 저장
Iterator it = c.Iterator();
// 읽어 올 요소가 남아있는지 확인
while(it.hasNext()) {
    // 있다면 다음 요소를 읽어온다.
    System.out.println(it.next());
}
```

### 2.1 Iterator인터페이스의 메서드
| 메서드 | 설명 |
|---|---|
| boolean hasNext() | 읽어 올 요소가 남아있는지 확인 (true/false) |
| Object next() | 다음 요소를 읽어온다. </br> next()를 호출하기 전에 hasNext()를 호출해서 읽어 올 요소가 있는지 확인하는 것이 안전하다. |
| void remove() | next()로 읽어 온 요소를 삭제한다. </br> next()를 호출한 다음에 remove()를 호출해야한다. (선택적 기능) |
| void forEachRemaining(Consumer<?super E> action) | 컬렉션에 남아있는 요소들에 대해 지정된 작업(action)을 수행한다. </br> 람다식을 사용하는 default 메서드(jdk1.8 추가) |

### 2.2 Map과 Iterator
- Map에는 iterator()가 없기때문에 `keySet()`, `entrySet()`,`values()`와 같이 </br> 
반환타입이 `Set`, `Collection`인 메서드를 이용해여 따로 Set과 Collection타입으로 호출한 후 다시 iterator()를 이용한다.
```java
Map map = new HashMap();
Iterator it = map.entrySet().iterator();
```

## 3. ListIterator
- Iterator를 상속받아 `기능`을 `추가`한 것이다
- 컬렉션의 요소에 접근할 때 Iterator는 `단방향(next)`으로만 이동할 수 있지만 ListIterator는 `양방향(next, previous)`으로 이동 가능하다.
- `listIterator()`를 통해서 얻을 수 있다. (List를 구현한 컬렉션 클래스에 존재)

### 3.1 ListIterator의 메서드
| 메서드 | 설명 |
|---|---|
| boolean hasNext() | 읽어 올 다음 요소가 남아있는지 확인 (true/false) |
| boolean hasPrevious() | 읽어 올 이전 요소가 남아있는지 확인 (true/false) |
| Object next() | 다음 요소를 읽어 온다. </br> next()를 호출하기 전에 hasNext()를 호출해서 읽어 올 요소가 있는지 확인하는 것이 안전하다. |
| Object previous() | 이전 요소를 읽어 온다. </br> previous()를 호출하기 전에 hasPrevious()를 호출해서 읽어 올 요소가 있는지 확인하는 것이 안전하다. |
| int nextIndex() | 다음 요소의 index를 반환 |
| int previousIndex() | 이전 요소의 index를 반환 |
| void add(Object o) | 컬렉션에 새로운 객체 추가(선택적 기능) |
| void remove() | 읽어온 요소 삭제 </br> 반드시 next()나 previous()를 먼저 호출한 후 호출(선택적 기능) |
| void set(Object o) | 읽어온 요소를 지정된 객체로 변경 </br> 반드시 next()나 previous()를 먼저 호출한 후 호출(선택적 기능) |

## 4. 선택적 기능
- 선택적 기능은 반드시 구현할 필요가 없다는 뜻이다.
```java
// 선택적 기능 예

// 예외를 던져서 구현되지 않는 기능이라는 것을 호출하는 쪽에 알린다.
public void remove() {
    throw new UnsupportedOperationException();
}

// 단순히 public void remove() {};라고 하는 것보다 예외를 던지는 것이 좋다.
```

# Reference
> 자바의 정석 - 남궁성

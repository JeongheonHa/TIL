# ArrayList & LinkedList

## 1. ArrayList
- 컬렉션 프레임웍에서 가장 많이 사용되는 컬렉션 `클래스`
- List인터페이스를 구현하기 때문에 데이터의 `저장순서가 유지`되고 `중복을 허용`한다.
- `ArrayList`는 기존의 `Vector`를 개선한 것으로 구현원리와 기능은 동일 (ArrayList를 사용하도록 하자 !)
- Vector는 자체적으로 동기화 처리가 되어 있으나, ArrayList는 동기화 처리가 되어있지 않다.
- 데이터의 저장공간으로 `배열`을 사용한다.

```java
public class Vector extends AbstractList implements List, RandomAccess, Cloneable, java.io.Serializable {
    ...
    // 객체를 담기위한 배열
    protected Object[] elementData; 
    
    // 선언된 객체의 타입이 모든 객체의 최고 조상인 `Object[]`이므로 모든 종류의 객체를 담을 수 있다.
}
```

### 1.1 ArrayList의 생성자와 메서드
- 생성자

| 생성자 | 설명 |
|---|---|
| ArrayList() | 크기가 10인 ArrayList를 생성 |
| ArrayList(Collection c) | 주어진 컬렉션이 저장된 ArrayList를 생성 |
| ArrayList(int initialCapacity) | 지정된 초기용량을 갖는 ArrayList를 생성 |

- 메서드

| 인터페이스 | 매서드 | 설명 |
|---|---|---|
| C | boolean add(Object o) | ArrayList의 마지막에 객체를 `추가` (성공하면 true, 실패하면 false) |
| L | void add(int index, Object element) | 지정된 위치에 `객체`(element)를 `저장` |
| C | boolean addAll(Collection c) | 주어진 컬렉션의 `모든 객체`를 `저장` (true/false) |
| L | boolean addAll(int index, Collection c) | 지정된 위치부터 주어진 컬렉션의 `모든 객체`를` 저장` (true/false) |
| C | void clear() | ArrayList를 완전히 `비운다`. | 
|| Object clone() | ArrayList를 `복제` |
| C | boolean contains(Object o) | 지정된 객체(o)가 ArrayList에 포함되어 있는지 `확인` (true/false) |
|| void ensureCapacity(int minCapacity) | ArrayList의 `용량`이 최소한 minCapacity가 되도록 한다. |
| L | Object get(int index) | 지정된 위치에 저장된 `객체`를 `반환` |
| L | int indexOf(Object o) | 지정된 객체가 저장된 `위치`를 찾아 `반환` (`-->`) |
| C | boolean isEmpty() | ArrayList가 비어있는지 `확인` (true/false) |
| C | Iterator iterator() | ArrayList의 `Iterator객체`를 `반환` |
| L | int lastIndexOf(Object o) | 객체가 저장된 `위치`를 `반환` (`<--`) |
| L | ListIterator listIterator() | ArrayList의 `ListIterator`를 `반환` |
| L | ListIterator listIterator(int index) | ArrayList의 지정된 위치부터 시작하는 `ListIterator`를 `반환` |
| L | Object remove(Object o) | 지정된 위치에 있는 `객체`를 `제거` |
| C | boolean remove(Object o) | 지정된 `객체`를 `제거` (true/false) |
| C | boolean removeAll(Collection c) | 지정된 컬렉션에 저장된 것과 `동일한 객체`들을 ArrayList에서 `제거` (true/false) |
| C | boolean retainAll(Collection c) | ArrayList에 저장된 객체 중에서 주어진 컬렉션과 `공통된 것들만을 남기고` </br> 나머지는 `삭제` (true/false) |
| L | Object set(int index, Object element) | 주어진 `객체`(element)를 지정된 위치에 `저장`하여 바꿈 |
| C | int size() | ArrayList에 저장된 `객체의 개수`를 `반환` |
| L | void sort(Comparator c) | 지정된 정렬기준(c)으로 ArrayList를 `정렬` |
| L | List subList(int fromIndex, int toIndex) | fromIndex부터 toIndex사이에 저장된 `객체`를 `반환` |
| C | Object[] toArray() | ArrayList에 저장된 모든 객체들을 `객체배열`로 `반환` |
| C | Objet[] toArray(Object[] a) | ArrayList에 저장된 모든 객체들을 `객체배열` a에 담아 `반환` |
|| void trimToSize() | 용량을 크기에 맞게 줄인다 (`빈공간을 없앤다.`) |
> C : collection인터페이스, L : List인터페이스

### 1.2 ArrayList에 저장된 객체의 삭제과정
- ArrayList에 중간에 저장된 데이터를 제거할 경우
    + 삭제할 데이터 다음의 데이터를 한 칸씩 이전 데이터에 복사해서 삭제할 데이터를 덮어쓴다. (1)
    + 데이터가 모두 한 칸씩 이동했으므로 마지막 데이터는 null로 변경한다. (2)
    + 데이터가 삭제되어 데이터의 개수가 줄었으므로 size의 값을 감소시킨다. (3)   
> - 마지막 데이터를 삭제할 경우 1번 과정은 필요없다.   
> - 중간에 저장된 데이터를 제거하면 배열 복사가 발생하여 부담이 많이 되며 되도록 1번 과정이 발생하지 않도록 하자 !!
### 1.3 ArrayList의 장단점
- 장점 
    + 배열은 `구조`가 `간단`하고 데이터를 읽는 시간(`접근시간`)이 `짧다`.   
    : index가 n인 데이터의 주소 = `배열의 주소` + `n * 데이터 타입의 크기`
- 단점
    + 실행 중 크기를 변경 x
    + `비순차적인`(중간에) 데이터의 `추가`, `삭제`에 시간이 많이 걸린다. (데이터를 하나씩 모두 옮기므로)
- 크기를 변경해야 하는 경우
    - 순서 : 더 큰 배열 생성 -> 배열 복사 -> 참조 변수 변경
    - 크기를 변경하기 위해 충분히 큰 배열을 생성하면 `메모리`가 `낭비`된다.
> 단, 순차적 데이터 추가 or 삭제는 빠르다.

## 2. LinkedList
- 배열과 달리 LinkedList는 `불연속적`으로 존재하는 데이터를 `연결`한다.
- 배열의 단점인 `크기 변경`, 비순차적 데이터의 `추가/삭제 시간`을 `보완`
- LinkedList의 각 요소를 `node`라고 하며 node에는 `다음 주소`와 `데이터`로 구성된다.
```java
class Node {
    // 다음 node의 주소를 참조변수 next에 저장
    Node next;
    // node에 저장된 데이터
    Object obj;
}
```

### 2.1 LinkedList의 메서드
- 생성자

| 생성자 | 설명 |
|---|---|
| LinkedList() | LinkedList객체 생성 |
| LinkedList(Collection c) | 주어진 컬렉션을 포함하는 LinkedList객체 생성 |

- LinkedList의 메서드 1 (collection, List인터페이스를 구현)

| 인터페이스 | 매서드 | 설명 |
|---|---|---|
| C | boolean add(Object o) | LinkedList의 마지막에 객체를 `추가` (성공하면 true, 실패하면 false) |
| L | void add(int index, Object element) | 지정된 위치에 `객체`(element)를 `저장` |
| C | boolean addAll(Collection c) | 주어진 컬렉션의 `모든 객체`를 LinkedList 끝에 `저장` (true/false) |
| L | boolean addAll(int index, Collection c) | 지정된 위치부터 주어진 컬렉션의 `모든 객체`를` 저장` (true/false) |
| C | void clear() | LinkedList를 완전히 `비운다`. | 
| C | boolean contains(Object o) | 지정된 객체(o)가 LinkedList에 포함되어 있는지 `확인` (true/false) |
| L | Object get(int index) | 지정된 위치에 저장된 `객체`를 `반환` |
| L | int indexOf(Object o) | 지정된 객체가 저장된 `위치`를 찾아 `반환` (`-->`) |
| C | boolean isEmpty() | LinkedList가 비어있는지 `확인` (true/false) |
| C | Iterator iterator() | `Iterator`를 `반환` |
| L | int lastIndexOf(Object o) | 객체가 저장된 `위치`를 `반환` (`<--`) |
| L | ListIterator listIterator() | `ListIterator`를 `반환` |
| L | ListIterator listIterator(int index) | 지정된 위치부터 시작하는 `ListIterator`를 `반환` |
| L | Object remove(Object o) | 지정된 위치에 있는 `객체`를 `제거` |
| C | boolean remove(Object o) | 지정된 `객체`를 `제거` (true/false) |
| C | boolean removeAll(Collection c) | 지정된 컬렉션에 저장된 것과 `동일한 객체`들을 모두 `제거` (true/false) |
| C | boolean retainAll(Collection c) | 주어진 컬렉션의 모든 요소가 `포함`되어 있는지 `확인` (true/false) |
| L | Object set(int index, Object element) | 주어진 `객체`(element)를 지정된 위치에 `저장`하여 바꿈 |
| C | int size() | LinkedList에 저장된 `객체의 개수`를 `반환` |
| L | List subList(int fromIndex, int toIndex) | fromIndex부터 toIndex사이에 저장된 `객체`를 List로 `반환` |
| C | Object[] toArray() | LinkedList에 저장된 모든 객체들을 `객체배열`로 `반환` |
| C | Objet[] toArray(Object[] a) | LinkedList에 저장된 모든 객체들을 `객체배열` a에 담아 `반환` |
> C : collection인터페이스, L : List인터페이스   
> ArrayList에서 Object clone(), void ensureCapacity(int minCapacity), void sort(Comparator c), void trimToSize() 없음   
> ArrayList 메서드의 종류와 기능은 거의 같다.

- LinkedList의 메서드 2

| 메서드 | 설명 |
|---|---|
| Object element() | LinkedList의 `첫 번째` 요소를 `반환` |
| boolean offer(Object o) | 지정된 객체를 LinkedList의 `끝`에 `추가` (true/false) |
| Object peek() | LinkedList의 `첫 번째` 요소를 `반환` |
| Object poll() | LinkedList의 `첫 번째` 요소를 `반환`, LinkedList에서는 `제거` |
| Object remove() | LinkedList의 `첫 번째` 요소 `제거` |
| void addFirst(Object o) | LinkedList의 `맨 앞`에 객체(o)를 `추가` |
| void addLast(Object o) | LinkedList의 `맨 뒤`에 객체를 `추가` |
| Iterator descendingIterator() | 역순으로 조회하기 위한 DescendingIterator를 `반환` |
| Object getFirst() | LinkedList의 `첫 번째` 요소를 `반환` |
| Object getLast() | LinkedList의 `마지막` 요소를 `반환` |
| boolean offerFirst(Object o) | LinkedList의 `맨 앞`에 객체를 `추가` 성공하면 true, 실패하면 false |
| boolean offerLast(Object o) | LinkedLust의 `맨 뒤`에 객체를 `추가` (true/false) |
| Object peekFirst() | LinkedList의 `첫 번째` 요소를 `반환` |
| Object peekLast() | LinkedList의 `마지막` 요소를 `반환` |
| Object pollFirst() | LinkedList의 `첫 번째` 요소를 `반환`하면서 `제거` |
| Object pollLast() | LinkedList의 `마지막` 요소를 `반환`하면서 `제거` |
| Object pop() | removeFirst() 와 동일 |
| void push(Object o) | addFirst() 와 동일 |
| Object removeFirst() | LinkedList의 `첫 번째` 요소 `제거` |
| Object removeLast() | LinkedList의 `마지막` 요소 `제거` |
| boolean removeFirstOccurrence(Object o) | LinkedList에서 `첫 번째로 일치`하는 객체를 `제거` |
| boolean removeLastOccurrence(Object o) | LinkedList에서 `마지막으로 일치`하는 객체를 `제거` |

### 2.2 LinkedList의 장단점
- 장점
    + 비순차적인 데이터의 `삭제`
    : 노드에 연결되어있는 주소만 바꿔주면 나중에 garbage collector가 제거해준다.
    + 비순차적인 데이터의 `추가`
    : 하나의 node객체를 생성하고 해당 주소를 다른 두 node에 연결해주면 된다.
    + node만을 추가 시키면 되기때문에 `크기변경`이 가능하다.
- 단점
    + `접근시간`(access time)이 느리다.
    : 불연속적인 요소들이 연결된 것이므로 처음부터 n까지의 node를 모두 들려야한다.

### 2.3 이중 연결 리스트
- `이중 연결리스트`(doubly linked list)   
: 이전에는 앞의 node밖에 알지 못했지만 이중 연결리스트는 앞뒤의 node를 알기 때문에 바로 전의 node로 이동 가능
```java
class Node {
    // 다음 node의 주소를 저장
    Node next;
    // 이전 node의 주소를 저장
    Node previous;
    Object obj;
}
```
> Java에서는 이중 연결리스트로 구현되어있다.

- `이중 원형 연결리스트`(doubly circular linked list)   
: 끝 node에서 처음 node로, 처음 node에서 끝 node로 이동 가능

## 3. ArrayList vs LinkedList
| 컬렉션 | 접근시간(읽기) | 추가/삭제 | 비고 |
|---|---|---|---|
| ArrayList | 빠르다 | 느리다 | 순차적인 추가삭제는 빠름 </br> 비효율적인 메모리 사용 |
| LinkedList | 느리다 | 빠르다 | 데이터가 많을수록 접근성이 떨어짐 |

# Reference
> 자바의 정석 - 남궁성

# Stack & Queue

## 1. Stack
- `LIFO`(Last In First Out)구조, 나중에 들어간게 처음으로 나온다.   
- 예시 : 수식계산, 수식괄호검사, 웹브라우저의 앞 뒤로 넘기는 기능
- Stack은 `클래스`이다.
```java
// 클래스이기 때문에 인스턴스를 생성할 수 있다.
Stack st = new Stack();
```

### 1.1 Stack의 메서드
| 메서드 | 설명 |
|---|---|
| boolean empty() | Stack이 비었는지 `확인` |
| Object peek() | Stack의 `맨 뒤`에 저장된 객체를 `반환`, pop()과 달리 Stack에서 객체를 꺼내지 않는다. </br> 비었을 경우 EmptyStackException발생) |
| Object pop() | Stack의 `맨 뒤`에 저장된 객체를 꺼낸다.(`삭제`) </br> 비었을 경우 EmptyStackException발생) |
| Object push(Object item) | Stack에 객체(item)를 `저장` |
| int search(Object o) | Stack에서 주어진 객체를 찾아서 그 `위치`를 `반환` (못찾으면 -1 반환) |

## 2. Queue
- `FIFO`(First In First Out)구조, 처음에 들어간게 처음으로 나온다.
- 예시 : 최근 사용문서, 버퍼, 맛집 줄서는 상황
- Queue는 `인터페이스`이기 때문에 인스턴스를 생성할 수 없다.

### 2.1 Queue의 메서드
| 메서드 | 설명 |
|---|---|
| boolean add(Object o) | 지정된 객체를 Queue에 `추가` </br> 성공하면 true, 저장공간이 부족하면 `lllegalStateException` 발생 |
| Object remove() | Queue에서 객체를 꺼내 `반환`,`삭제` </br> 비어있으면 `NoSuchElementException` 발생 |
| Object element() | 삭제없이 요소를 읽어온다(`반환`) </br> peek과 달리 Queue가 비었을 때 `NoSuchElementException` 발생 |
| boolean offer(Object o) | Queue에 객체를 `저장` </br> 성공하면 `true`, 실패하면 `false` |
| Object poll() | Queue에서 객체를 꺼내서 `반환`, `삭제` </br> 비어있으면 `null` 반환 |
| Object peek() | 삭제없이 요소를 읽어온다(`반환`) </br> Queue가 비어있으면 `null` 반환 | 

### 2.2 Queue인터페이스 사용방법
- Queue 직접구현
- Queue가 구현된 클래스 사용 (`다형성`을 이용)
    + Queue가 구현된 클래스 : AbstractQueue, ArrayBlockingQueue, ArrayDeque, ConcurrentLinkedDeque, ConcurrentLinkedQueue,   
    DelayQueue, LinkedBlockingDeque, LinkedBlockingQueue, `LinkedList`, LinkedTransferQueue, PriorityBlockingQueue,   
    PriorityQueue, SynchronousQueue
```java
Queue q = new LinkedList();
q.offer("0");
```

### 2.3 Queue의 변형
- PriorityQueue
    + 저장한 순서에 상관없이 `우선순위`가 높은 것부터 꺼낸다. (null 저장 x)   
    + null을 저장할 경우 NullPointerException 발생
    + 저장공간으로 `배열` 사용
    + 각 요소를 `heap`이라는 자료구조의 형태로 저장
    + 숫자의 경우 1,2,3,4, ... 순, 객체의 경우 비교하는 방법을 정의해야한다.

- Deque(Double-Ended Queue)
    + `양쪽 끝`에서 저장(offer)과 삭제(poll) 가능
    + Deque의 조상은 Queue이며 구현체는 ArrayDeque, LinkedList ... 가 있다.
    + Deque는 Stack과 Queue를 하나로 합쳐놓은 것과 같으며 스택으로 사용할 수도 있고 큐로 사용할 수도 있다.
    
# Reference
> 자바의 정석 - 남궁성

# 스택과 큐

## ✅ 스택(Stack)
선형 자료구조의 일종으로 `LIFO`(Last In First Out)형식의 자료구조이다.
</br>
</br>

<img src = "https://user-images.githubusercontent.com/108064146/192323986-ce77cc34-80da-47cf-bb8c-4857a625e408.png">

### 스택의 특징
- 스택은 아래에서 위로 쌓이는 형식이며 가장 최근에 들어온 데이터를 `top`이라고 부른다.
- 스택은 자료의 삽입과 삭제가 한 곳(top)에서만 이루어진다.
- 스택이 비어있을 경우 데이터를 꺼내려고하면 `스택 언더플로우`(Stack Underflow)가 발생하고 스택이 꽉 차있을 때 데이터를 삽입하려하면 `스택 오버플로우`(Stack Overflow)가 발생한다.

### 스택의 연산
- `push(item)` : item 하나를 스택의 가장 윗 부분에 추가 - `O(1)`
- `pop()` : 스택에서 가장 위에 있는 항목을 제거 - `O(1)`
- `peek()` : 스택의 가장 위에 있는 항목을 반환 - `O(1)`
- `isEmpty()` : 스택이 비어있는지 확인 - `O(1)`

### 파이썬으로 스택 구현
파이썬에는 Stack을 제공하지 않기 때문에 만약 사용한다면 직접 구현하도록 하자.

```python
class Stack:
    def __init__(self):
        self.top = []
        
    def push(self, item):
        self.top.append(item)
            
    def pop(self):
        if not self.isEmpty():
            return self.top.pop(-1)
        else:
            print("Stack is empty")
            exit()
            
    def peek(self):
        if not self.isEmpty():
            return self.top[-1]
        else:
            print("Stack is empty")
            exit()
    
    def isEmpty(self):
        return len(self.top) == 0

    def __len__(self):
        return len(self.top)
    
    def __str__(self):
        return str(self.top[::1])
    
    def isContain(self, item):
        return item in self.top
    
    def size(self):
        return len(self.top)
    
    def clear(self):
        self.top = []
```

### 스택의 사용 사례
- 재귀 알고리즘
    + 재귀적으로 함수를 호출해야 하는 경우에 임시 데이터를 스택에 넣어준다.
    + 재귀함수를 빠져나와 백트래킹을 할 때는 스택에 넣어 두었던 임시 데이터를 빼 줘야 한다.
    + 스택은 이런 일련의 행위를 직관적으로 가능하게 해준다.
    + 스택은 재귀 알고리즘을 반복적 형태를 통해서 구현할 수 있게 해준다.
- 역순 문자열 만들기 (맨 끝의 문자열부터 차례대로 만든다.)
- 수식의 괄호 검사
- 후위 표기법 계산
- 웹 브라우저 뒤로가기
- 문서작업에서 Ctrl+Z
- 실행 취소(undo)

## ✅ 큐(Queue)
선형 자료구조의 일종으로 `FIFO`(First In First Out)형식의 자료구조이다.   

📎 참고로 Java Colection에서 Queue는 인터페이스이며 이를 구현한 Priority queue등을 사용할 수 있다.
</br>
</br>

<img src = "https://user-images.githubusercontent.com/108064146/192325725-974285f1-82c0-40ef-8f8b-629dac315dcf.png">

### 큐의 특징
- 정해진 한 곳(top)에서만 삽입, 삭제가 이루어지는 스택과는 달리 큐는 **Rear부분에서 데이터의 삽입**이 이루어지고 **Front부분에서 데이터의 삭제**가 이루어진다.
- 큐의 Rear부분에서 이루어지는 삽입연산을 `enQueue`(인큐), Front부분에서 이루어지는 삭제연산을 `deQueue`(디큐)라고 한다.

### 큐의 연산
- `enqueue(data)` : rear에 data를 삽입
- `dequeue()` : front에서 data 삭제
- `peek()` : front부분의 data를 꺼내지 않고 어떤 값인지 반환
- `isEmpty()` : 큐가 비어있는지 확인


### 파이썬으로 큐 구현
큐는 리스트로 구현하는 것도 가능하지만 효율적이지 않다. 리스트의 맨 처음에 원소를 추가하거나 뺄 경우 다른 원소들을 모두 한 칸씩 `shift`해야하기 때문이다. 따라서, `collections.deque`사용하거나 파이썬에서 제공하는 `Queue모듈`을 이용하거나 `Linked List`를 활용하는 것이 좋다. 

#### 리스트로 구현 : 
```python
class Queue():
    def __init__(self):
        self.queue = []
        
    def enqueue(self, data):
        self.queue.append(data)
        
    def dequeue(self):
        dequeue_object = None
        if self.isEmpty():
            print("Queue is Empty")
        else:
            dequeue_object = self.queue[0]
            self.queue = self.queue[1:]
            
        return dequeue_object
            
    def peek(self):
        peek_object = None
        if self.isEmpty():
            print("Queue is Empty")
        else:
            peek_object = self.queue[0]
            
        return peek_object
            
    def isEmpty(self):
        is_empty = False
        if len(self.queue) == 0:
            is_empty = True
        return is_empty
```
#### Linked List로 구현 :

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None

class LinkedListQueue():
    def __init__(self):
        self.front = None
        self.rear = None
        
    def enqueue(self, data):
        new_node = Node(data)
        
        if self.front is None:
            self.front = new_node
            self.rear = new_node
        else:
            self.rear.next = new_node
            self.rear = self.rear.next
        
    def dequeue(self):
        dequeue_object = None
        if self.isEmpty():
            print("Queue is Empty")
        else:
            dequeue_object = self.front.data
            self.front = self.front.next
            
        if self.front is None:
            self.rear = None
        return dequeue_object
    
    def peek(self):
        front_object = None
        if self.isEmpty():
            print("Queue is Empty")
        else:
            front_object = self.front.data            
        return front_object
    
    def isEmpty(self):
        is_empty = False
        if self.front is None:
            is_empty = True
        return is_empty
```
### 큐의 사용 사례
- 우선순위가 같은 작업 예약(프린터의 인쇄 대기열)
- 프로세스 관리(스케줄링)
- 너비 우선 탐색(BFS) 구현
- 캐시(Cache) 구현

## References
- <https://cocoon1787.tistory.com/691>
- <https://daimhada.tistory.com/107>
- <https://somjang.tistory.com/entry/파이썬으로-구현하는-자료구조-큐-Queue>

## ✅ 양방향 연결 리스트(Doubly Linked List)

노드들이 양쪽 방향으로 연결된 리스트를 말한다.

- 단방향 연결 리스트의 단점인 이전 노드를 알기 위해선 head노드부터 차례로 탐색을 해야한다는 점을 보완했다.
- **이전 노드로의 링크(prev)를 포함**해 왼쪽으로도 이동이 가능하다.
- 마지막 노드와 첫 노드가 서로 연결된 **원형 리스트**를 가정한다.
- **head노드**는 항상 **dummy노드**가 되도록 한다.   

📎 dummy node : 리스트의 처음을 구분할 수 있게 "**marker**"의 기능을 하는 특별한 노드로 빈 리스트는 dummy node 하나로 이루어진다.</br>

<img src = "https://user-images.githubusercontent.com/108064146/193051242-c76eb547-a4d9-439a-bac0-76cc259e5e3a.jpeg">

## ✅ 양방향(원형) 연결 리스트의 연산
- `moveAfter(a, x)` : a노드를 x노드 뒤로 이동 - `O(1)`
- `moveBefore(a, x)` : a노드를 x노드 앞으로 이동 - `O(1)`
- `insertAfter(x, key)` : 데이터가 key인 노드를 x노드 뒤에 삽입 - `O(1)`
- `insertBefore(x, key)` : 데이터가 key인 노드를 x노드 앞에 삽입 - `O(1)`
- `pushFront(key)` : 데이터가 key인 노드를 head 다음에 추가 - `O(1)`
- `pushBack(key)` : 데이터가 key인 노드를 head 이전에 추가 - `O(1)`
- `remove(x)` : 리스트에서 x노드를 제거, 없으면 None - `O(1)`
- `popFront()` : head 다음에 있는 노드를 제거 후 데이터 값 리턴, 없으면 None - `O(1)`
- `popBack()` : head 이전에 있는 노드를 제거 후 데이터 값 리턴, 없으면 None - `O(1)`
- `search(key)` : 데이터 값이 key인 노드를 찾아서 반환, 없으면 None - `O(n)`

## ✅ 노드 클래스

```python
class Node:
    def __init__(self, key = None):
        self.key = key
        self.next = None
        self.prev = None
```

## ✅ 양방향(원형) 연결 리스트 클래스

```python
class DoublyLinkedList:
    def __init__(self):
        self.head = Node()
        self.size = 0
    
    def __iter__(self):
        v = self.head.next
        while v != self.head:
            yield v
            v = v.next

    def __len__(self):
        return self.size

    def __str__(self):
        return "->".join(str(v) for v in self)

```

## ✅ splice 메서드
다른 연산에 사용되는 기본적인 메서드</br>

<img src = "https://user-images.githubusercontent.com/108064146/193051460-6fff08fd-429c-49c3-b2d6-00e90295ccd5.jpeg"></br>

### splice의 조건
- a 노드 다음에 b 노드가 존재한다. (바로 옆이 아니어도 상관없음)
- a 노드와 b 노드 사이에 head(더미) 노드가 존재하지 않는다.
- a 노드와 b 노드 사이에 x 노드가 존재하지 않는다.

```python
def splice(self, a, b, x):
    if a == None or b == None or x == None:
        return None
    
    ap = a.prev
    bn = b.next
    
    # cut [a:b]
    ap.next = bn
    bn.prev = ap
    
    # insert [a:b] after x
    xn = x.next
    xn.prev = b
    b.next = xn
    a.prev = x
    x.next = a
```

## ✅ 이동과 삽입

```python
def moveAfter(self, a, x):        # x 뒤로 a 이동
    splice(a, a, x)
    
def moveBefore(self, a, x):       # x 앞으로 a 이동
    splice(a, a, x.prev)
    
def insertAfter(self, x, key):    # x 뒤에 key 삽입
    moveAfter(Node(key), x)
    
def insertBefore(self, x, key):   # x 앞에 key 삽입
    moveBefore(Node(key), x)

def pushFront(self, key):         # 맨 앞에 key 추가
    insertAfter(self.head, key)

def pushBack(self, key):          # 맨 뒤에 key 추가
    insertBefore(self.head, key)
```

## ✅ 삭제

```python
def remove(self, x):
    if x == None or x == self.head:
        return None
    x.prev.next = x.next
    x.next.prev = x.prev

def popFront(self):
    if self.size == 0:
        return None
    key = head.next.key
    self.remove(head.next)
    return key

def popBack(self):
    if self.size == 0:
        return None
    key = head.prev.key
    self.remove(head.prev)
    return key
```

## ✅ 탐색

```python
def search(self, key):
    v = self.head.next
    while v != self.head:
        if v.key == key:
            return v
        v = v.next
    return None
```

## References
https://www.youtube.com/watch?v=zWrFVf9_YTQ&list=PLsMufJgu5933ZkBCHS7bQTx0bncjwi4PK&index=16

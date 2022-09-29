## ✅ 단방향 연결 리스트(Singly Linked List)
노드들이 한쪽 방향으로만 연결된 리스트를 말한다.

- 가장 앞에 있는 노드를 **head**노드라고 하며 **head**노드를 통해 리스트의 노드를 접근한다.
- **next**의 주소가 **None**이라면 그 노드가 마지막 노드이다.

<img src = "https://user-images.githubusercontent.com/108064146/192981126-6adc5338-fb2e-43a1-afa1-90526839b442.png">

## ✅ 단방향 연결 리스트의 특징
기존에 작성한 적이 있는 내용이므로 아래의 링크를 통해 보시면 됩니다.   
</br>
<https://jh2476.tistory.com/23#1.2.%20✅%20연결%20리스트(Linked%20List)>

## ✅ 단방향 연결 리스트의 연산
- `pushFront(key)` : key값을 갖는 새 노드를 self의 가장 앞에 삽입 - `O(1)`
- `pushBack(key)` : key값을 갖는 새 노드를 self의 가장 뒤에 삽입 - `O(n)`
- `popFront()` : self의 첫 노드를 삭제한 후 key값을 반환 - `O(1)`
- `popBack()` : self의 마지막 노드를 삭제한 후 key값을 반환 - `O(n)`
- `search(key)` : self에서 key값을 갖는 노드를 찾아 반환 - `O(n)`

## ✅ 노드 클래스

```python
class Node:
    def __init__(self, key=None):
        self.key = key
        self.next = None

    def __str__(self):
        return str(self.key)
```

## ✅ 단방향 연결 리스트 클래스

```python
class SinglyLinkedList:
    def __init__(self):
        self.head = None
        self.size = 0

    def __iter__(self):     # 제너레이터 정의
        v = self.head
        while v != None:
            yield v
            v = v.next
    
    def __str__(self):
        return "->".join(str(v) for v in self)

    def __len__(self):
        return self.size
```

## ✅ pushFront 메서드

<img src = "https://user-images.githubusercontent.com/108064146/192977732-83e0b2c7-b6fc-450d-865f-edf59d1373fd.jpeg"></br></br>

```python
def pushFront(self, key):
    new_node = Node(key)        # 노드 생성
    new_node.next = self.head   # 생성한 노드 앞뒤 참조 변경
    self.head = new_node
    self.size += 1              # 크기 증가
```

## ✅ pushBack 메서드

<img src = "https://user-images.githubusercontent.com/108064146/192978180-4d382871-3247-4d0c-b112-c736ac0a9411.jpeg"></br></br>

```python
def pushBack(self, key):
        new_node = Node(key)            # 노드 생성
        if self.size == 0:              # 노드가 없는 경우
            self.head = new_node
        else:                           # 노드가 있는 경우
            tail = self.head
            while tail.next != None:    # tail이 마지막 노드일 때 까지 순회
                tail = tail.next
            tail.next = new_node        # tail의 다음 노드로 새로 생성한 노드 지정
        self.size += 1                  # 크기 증가
```

## ✅ popFront 메서드

<img src = "https://user-images.githubusercontent.com/108064146/192978280-a347820b-7eca-4412-bfb8-0bd7540d21e5.jpeg"></br></br>

```python
def popFront(self):
        if self.size == 0:      # 노드가 없는 경우
            return None
        else:                   # 노드가 있는 경우
            x = self.head       # 참조 변수 x로 head 노드 참조
            key = x.key         # x의 key를 key참조변수로 따로 지정
            self.head = x.next  # head로 x의 다음 노드를 참조
            self.size -= 1      # 크기 감소
            
            del x               # 참조 변수 x가 참조하는 노드 제거
            return key
```

## ✅ popBack 메서드

<img src = "https://user-images.githubusercontent.com/108064146/192978386-67d83129-992b-4457-9d60-698dd7b73069.jpeg"></br></br>

```python
def popBack(self):
        if self.size == 0:                  # 노드가 없는 경우 (1)
            return None
        else:
            prev, tail = None, self.head 
            while tail.next != None:        # tail의 다음 노드가 None일 때 까지 순회
                prev = tail
                tail = tail.next
                if prev == None:            # 노드가 1개인 경우 (2)
                    self.head = None
                else:                       # 노드가 2개 이상인 경우 (3)
                    prev.next = tail.next
                key = tail.key              # 제거 과정
                del tail
                self.size -= 1
                return key
```

## ✅ search 메서드

```python
def search(self, key):
    v = self.head
    while v != None:
        if v.key == key:
            return v
        v = v.next
```

## References
<https://www.youtube.com/watch?v=aCHwXmpuAkY&list=PLsMufJgu5933ZkBCHS7bQTx0bncjwi4PK&index=14>

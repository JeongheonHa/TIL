# 배열과 연결 리스트의 차이

## ✅ 배열(Array)
순차적으로 저장되고 고정된 크기를 갖는 자료구조이다. 

</br>
<img src = "https://user-images.githubusercontent.com/108064146/192277653-8f6a2551-eb4b-4c44-8089-19eb1c34709f.jpeg">
</br>

### 배열의 특징
- 구조가 단순하다.

- 원하는 값의 인덱스를 `random access`로 접근하여 `O(1)`의 시간복잡도를 가진다.

- `Cache Hit Rate`가 높다.

- `Compile time`에 할당되는 정적 메모리 할당

### 배열의 한계
- 비순차적인(중간에) 원소의 추가, 삭제의 경우 빈 공간 또는 추가한 원소에 의해 나머지 원소들을 한 칸씩 `shift`해줘야하는 비용이 발생하므로 시간이 많이 걸리며 최악의 경우 시간 복잡도는 `O(n)`이 된다. 

- 크기를 변경할 수 없어 `더 큰 배열 생성 -> 배열 복사 -> 참조 변수 변경`의 순서로 크기를 변경해야하며 크기를 변경하기 위해 충분히 큰 배열을 생성하면 `메모리가 낭비`된다.

## ✅ 연결 리스트(Linked List)
배열의 고정된 크기와 비순차적인 추가, 삭제의 단점을 보완한 자료구조이다.

</br>
<img src = "https://user-images.githubusercontent.com/108064146/192277913-1bf341e5-24a1-47d5-99b7-c8f7ba5164df.jpeg">
</br>

### 연결 리스트의 특징
- 데이터 적재에 빈틈이 없다.

- 각각의 원소는 자신 다음 원소의 주소를 알고 있기 때문에 해당 주소만 다른 데이터로 변경함으로써 크기를 변경할 수 있고 비순차적인 추가, 삭제에 `O(1)`의 시간복잡도를 가질 수 있다.

- Tree 구조에 유용하게 사용된다.

- 새로운 node가 추가되는 `Runtime`에 할당되는 동적 메모리 할당

### 연결 리스트의 한계
- 순차적이지 않아 `cash hit`이 어렵다.

- 원하는 위치에 추가, 삭제를 할 때 search를 하는 과정에서 첫 번째 원소부터 원하는 원소의 위치까지 모두 확인해봐야하기 때문에 이러한 과정에서 시간복잡도가 `O(n)`이 된다.


## References
- <https://ongveloper.tistory.com/403>
- <https://jy-tblog.tistory.com/38>
- <https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/master/DataStructure>

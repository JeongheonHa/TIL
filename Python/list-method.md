# List(리스트) 메서드
📎 `메서드 -> 타입`은 반환 타입을 적어 놓은 것입니다.

## ✅ list.append(elem) -> None
리스트의 마지막에 요소을 추가한 후 None 반환

```python
>>> fruits = ['apple', 'banana', 'cherry']
>>> fruits.append("orange")
>>> print(fruits)
```
```python
# 결과
['apple', 'banana', 'cherry', 'orange']
```

📎 elem : Required. An element of any type (string, number, object etc.)

## ✅ list.clear() -> None
리스트의 모든 요소를 삭제 (`del a[:]`와 같다.)

```python
>>> fruits = ['apple', 'banana', 'cherry', 'orange']
>>> fruits.clear()
>>> print(fruits)
```
```python
# 결과
[]
```
## ✅ list.copy() -> list
리스트의 얕은 사본을 반환 (`a[:]`와 같다.)

```python
>>> fruits = ['apple', 'banana', 'cherry', 'orange']
>>> print(fruits.copy())
```
```python
# 결과
['apple', 'banana', 'cherry', 'orange']
```
## ✅ list.count(value) -> int
리스트에서 지정된 값(value)이 몇 개 있는지 반환

```python
fruits = ['apple', 'banana', 'cherry']
print(fruits.count("cherry"))
```
```python
# 결과
1
```
## ✅ list.extend(iterable) -> list
리스트의 마지막에 리스트의 요소(or any iterable)들을 덧붙여 확장한 리스트를 반환

```python
>>> fruits = ['apple', 'banana', 'cherry']
>>> cars = ['Ford', 'BMW', 'Volvo']
>>> print(fruits.extend(cars))
```
```python
# 결과
['apple', 'banana', 'cherry', 'Ford', 'BMW', 'Volvo']
```
📎 iterable : 반복 가능한 객체 (list, dict, set, str, bytes, tuple, range etc.)

## ✅ list.index(elem [, start[, end]]) -> int
지정된 값(elem)과 첫 번째로 일치하는 위치를 반환. 그런 요소가 없으면 ValueError 발생

```python
>>> fruits = ['apple', 'banana', 'cherry']
>>> print(fruits.index("cherry"))
```
```python
# 결과
2
```

선택적인 인자 start 와 end 는 슬라이스 표기법(`[start:end]`)처럼 해석되고, 검색을 리스트의 특별한 서브 시퀀스로 제한하는 데 사용된다. 돌려주는 인덱스는 start 인자가 아니라 전체 시퀀스의 시작을 기준으로 한다.

```python
>>> fruits = ['apple', 'banana', 'cherry']
>>> print(fruits.index("cherry",0,2)
```
```python
# 결과
ValueError: 'cherry' is not in list
```

📎 `[, start[, end]]`는 생략 가능하다는 것을 나타낼 때 사용되는 표기이다.

## ✅ list.insert(pos, elem) -> None
지정된 요소(elem)를 지정된 위치(pos)에 삽입한다.

```python
>>> fruits = ['apple', 'banana', 'cherry']
>>> fruits.insert(1, "orange")
>>> print(fruits)
```
```python
['apple', 'orange', 'banana', 'cherry']
```
## ✅ list.pop([pos]) -> elem의 타입
리스트에서 지정된 위치(pos)에 있는 요소를 제거하고 해당 요소를 반환. 

```python
>>> fruits = ['apple', 'banana', 'cherry']
>>> print(fruits.pop(1))
```
```python
# 결과
banana
```

인덱스를 지정하지 않으면, a.pop() 은 리스트의 마지막 요소를 제거하고 해당 요소를 반환. (pos의 기본값이 -1이기 때문에)

```python
>>> fruits = ['apple', 'banana', 'cherry']
>>> print(fruits.pop())
```
```python
# 결과
cherry
```

## ✅ list.remove(elem) -> None
리스트에서 지정된 값(elem)과 같은 첫 번째 요소를 제거. 요소가 없으면 ValueError 발생

```python
>>> fruits = ['apple', 'banana', 'cherry']
>>> fruits.remove("banana")
>>> print(fruits)
```
```python
# 결과
['apple', 'cherry']
```

## ✅ list.reverse() -> None
리스트의 요소들을 반대(내림차순)로 정렬한다.

```python
>>> fruits = ['apple', 'banana', 'cherry']
>>> print(fruits.reverse())
```
```python
# 결과
['cherry', 'banana', 'apple']
```

## ✅ list.sort(reverse=True|False, key=myFunc) -> None
리스트의 요소들을 오름차순으로 정렬한다.

```python
>>> cars = ['Ford', 'BMW', 'Volvo']
>>> cars.sort()
>>> print(cars)
```
```python
# 결과
['BMW', 'Ford', 'Volvo']
```

📎 `reverse`(선택 사항) : reverse=True를 할 경우 리스트를 내림차순으로 정렬한다. 기본값은 reverse=False   
📎 `key`(선택 사항) : 정렬의 기준을 정하는 함수를 key로 정한다. (lambda 가능)

</br>📌 key와 reverse의 사용 방법은 기존에 작성해놓은게 있어서 링크를 통해 확인하시면 됩니다.

<https://jh2476.tistory.com/11>

# References
- <https://docs.python.org/ko/3/tutorial/datastructures.html>
- <https://www.w3schools.com/python/python_ref_list.asp>

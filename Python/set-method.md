# set 메서드 

📎 `메서드 -> 타입`은 반환 타입을 적어 놓은 것입니다.

## ✅ set.add(elmnt) -> None
set에 요소를 추가한다.   
만약 요소가 존재할 경우 요소는 추가되지 않는다.
</br>

### Parameter Values
elmnt : Required. The element to add to the set
</br>

### Example

```python
thisset = {"apple", "banana", "cherry"}
thisset.add("apple")
print(thisset)  # {'banana', 'apple', 'cherry'}
```
</br>

## ✅ set.pop() -> type of elem
set에서 랜덤으로 요소 하나를 제거한 후 반환한다.
</br>

### Parameter Values
No parameter values.
</br>

### Example

```python
fruits = {"apple", "banana", "cherry"}
x = fruits.pop() 
print(x)    # cherry
```
</br>

## ✅ set.remove(item) -> None
set에서 특정 요소롤 제거한다. 만약 요소가 존재하지 않을 경우 error발생

📎 discard()와 기능은 동일하지만 요소가 존재하지 않으면 error가 발생한다.
</br>

### Parameter Values
item : Required. The item to search for, and remove
</br>

### Example

```python
fruits = {"apple", "banana", "cherry"}
fruits.remove("banana") 
print(fruits)   # {'apple', 'cherry'}
```
</br>

## ✅ set.discard(value) -> None
set에서 특정 요소를 제거한다. 만약 요소가 존재하지 않아도 error가 발생하지 않는다.

📎 remove()와 기능은 동일하지만 요소가 존재하지 않아도 error가 발생하지 않는다.
</br>

### Parameter Values
value : Required. The item to search for, and remove
</br>

### Example

```python
fruits = {"apple", "banana", "cherry"}
fruits.discard("banana") 
print(fruits)   # {'cherry', 'apple'}
```
</br>

## ✅ set.clear() -> None
set의 모든 요소를 제거한다.
</br>

### Parameter Values
No Parameters
</br>

### Example

```python
fruits = {"apple", "banana", "cherry"}
fruits.clear()
print(fruits)   # set()
```

## ✅ set.copy()
set의 복사본을 반환한다.
</br>

### Parameter Values
No parameters
</br>

### Example

```python
fruits = {"apple", "banana", "cherry"}
x = fruits.copy()
print(x)    # {'banana', 'apple', 'cherry'}
```
</br>

## ✅ set.update(set) -> None
다른 set또는 다른 iterable의 요소들을 추가함으로써 현재 set을 업데이트한다.

📎 만약 두 set에 요소가 존재할 경우 업데이트된 set에는 오직 한 개만 존재한다.
</br>

### Parameter Values
set	: Required. The iterable insert into the current set
</br>

### Example

```python
x = {"apple", "banana", "cherry"}
y = {"google", "microsoft", "apple"}
x.update(y) 
print(x)    # {'microsoft', 'apple', 'google', 'banana', 'cherry'}
```
</br>

## ✅ set.union(set1, set2...) -> set
여러 set의 합집합을 의미한다.
</br>

### Parameter Values
set1 : Required. The iterable to unify with
set2 : Optional. The other iterable to unify with. You can compare as many iterables as you like. (Separate each iterable with a comma)
</br>

### Example

```python
x = {"a", "b", "c"}
y = {"f", "d", "a"}
z = {"c", "d", "e"}
result = x.union(y, z)  # result = x | y | z와 같다.
print(result)   # {'c', 'd', 'e', 'a', 'f', 'b'}
```
</br>

## ✅ set.intersection(set1, set2 ... etc) -> set
여러 set의 교집합을 의미한다.
</br>

### Parameter Values
set1 : Required. The set to search for equal items in
set2 : Optional. The other set to search for equal items in. You can compare as many sets you like. (Separate the sets with a comma)
</br>

### Example

```python
x = {"a", "b", "c"}
y = {"c", "d", "e"}
z = {"f", "g", "c"}
result = x.intersection(y, z)   # result = x & y & z와 같다.
print(result)   # {'c'}
```
</br>

## ✅ set.difference(set) -> set
두 번째 set에 존재하지 않고 첫 번째 set에만 존재하는 요소들을 반환한다. (차집합을 의미한다.)
</br>

### Parameter Values
set	: Required. The set to check for differences in
</br>

### Example

```python
x = {"apple", "banana", "cherry"}
y = {"google", "microsoft", "apple"}
z = x.difference(y) # z = x - y와 같다.
print(z)    # {'banana', 'cherry'}
z = y.difference(x) # z = y - x와 같다. 
print(z)    # {'microsoft', 'google'}
```
</br>

## References
<https://www.w3schools.com/python/python_ref_set.asp>

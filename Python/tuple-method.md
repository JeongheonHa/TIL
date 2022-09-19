# tuple(튜플) 메서드

📎 `메서드 -> 타입`은 반환 타입을 적어 놓은 것입니다.

## ✅ tuple.count(value) -> int
튜플에 지정된 값의 등장한 횟수를 반환한다.

```python
thistuple = (1, 3, 7, 8, 7, 5, 4, 6, 8, 5)
print(thistuple.count(5))
```
```python
# 결과
2
```

## ✅ tuple.index(value) -> int
지정된 값이 처음 등장하는 위치를 반환한다.

📎 만약 값이 없을 경우 예외를 발생
</br>

```python
thistuple = (1, 3, 7, 8, 7, 5, 4, 6, 8, 5)
print(thistuple.index(8))
print(thistuple.index("고양이"))
```
```python
# 결과
3
ValueError: tuple.index(x): x not in tuple
```

# References
- <https://www.w3schools.com/python/python_ref_tuple.asp>

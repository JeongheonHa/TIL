# dictionary(딕셔너리) 메서드

📎 `메서드 -> 타입`은 반환 타입을 적어 놓은 것입니다.

## ✅ dictionary.clear() -> None
딕셔너리의 모든 요소를 제거한다.

```python
car = {"brand": "Ford", "model": "Mustang", "year": 1964}
car.clear()
print(car)
```
```python
# 결과
{}
```

## ✅ dictionary.copy() -> dict
해당 딕셔너리의 복사본을 반환한다.

```python
car = {"brand": "Ford", "model": "Mustang", "year": 1964}
print(car.copy())
```
```python
# 결과
{'brand': 'Ford', 'model': 'Mustang', 'year': 1964}
```

## ✅ dict.fromkeys(keys[, value]) -> dict
지정된 여러 키(keys)와 값(value)을 가진 새로운 딕셔너리를 반환한다.

```python
x = ('key1', 'key2', 'key3')
y = 0
print(dict.fromkeys(x, y))
```
```python
# 결과
{'key1': 0, 'key2': 0, 'key3': 0}
```

</br> 📎 keys(필수) :  딕셔너리에 넣을 key들   
📎 value(선택) : 모든 키에 대한 값, value를 지정하지 않을 경우 기본값은 모든 키에대한 값은 None이다.</br>

## ✅ dictionary.get(keyname[, value]) -> value의 타입
지정된 키(keyname)로 요소의 값(value)을 반환한다.

```python
car = {"brand": "Ford", "model": "Mustang", "year": 1964}
print(car.get("model"))
```
```python
Mustang
```

</br> 📎 딕셔너리에 존재하지 않는 키로 요소의 값을 반환하는 경우 지정된 값을 반환한다.(딕셔너리에는 변화가 없다.)

```python
car = {"brand": "Ford", "model": "Mustang", "year": 1964}
print(car.get("price", 15000))
print(car)
```
```python
# 결과
15000
{'brand': 'Ford', 'model': 'Mustang', 'year': 1964}
```

## ✅ dictionary.items() -> dict_items()
view object를 반환한다. view object는 딕셔너리의 모든 키-값 쌍들을 리스트 안의 튜플 형태로 포함한다.

📎 view object는 해당 딕셔너리의 모든 변경 사항을 반영한다.
</br>

```python
car = {"brand": "Ford", "model": "Mustang", "year": 1964}
x = car.items()
print(x)
# 기존 딕셔너리의 year키에 대한 값을 변경
car["year"] = 2018
print(x)
```
```python
# 결과
dict_items([('brand', 'Ford'), ('model', 'Mustang'), ('year', 1964)])
dict_items([('brand', 'Ford'), ('model', 'Mustang'), ('year', 2018)])
```

## ✅ dictionary.keys() -> dict_keys()
view object를 반환한다. view object는 딕셔너리의 모든 키를 리스트 형태로 포함한다.

📎 view object는 해당 딕셔너리의 모든 변경 사항을 반영한다.
</br>

```python
car = {"brand": "Ford", "model": "Mustang", "year": 1964}
x = car.keys()
print(x)
# 기존 딕셔너리에 키(color)와 값(white)를 저장
car["color"] = "white"
print(x)
```
```python
# 결과
dict_keys(['brand', 'model', 'year'])
dict_keys(['brand', 'model', 'year', 'color'])
```

## ✅ dictionary.pop(keyname[, defaultvalue]) -> value의 타입
딕셔너리에서 지정된 요소를 제거한 후 해당 요소의 값을 반환한다.

```python
car = {"brand": "Ford", "model": "Mustang", "year": 1964}
print(car.pop("model"))
print(car)
```
```python
# 결과
Mustang
{'brand': 'Ford', 'year': 1964}
```

## ✅ dictionary.popitem() -> tuple
딕셔너리에 가장 마지막으로 삽입된 요소를 제거한 후 해당 요소를 튜플 형태로 반환한다. (python 3.7 이전 버전에서는 무작위 요소를 제거)

```python
car = {"brand": "Ford", "model": "Mustang", "year": 1964}
print(car.popitem())
```
```python
# 결과
('year', 1964)
```

## ✅ dictionary.setdefault(keyname[, value]) -> value의 타입
지정된 키를 갖는 요소의 값을 반환한다.

📎 만약 키가 존재하지 않을 경우 지정된 값을 갖는 키를 삽입한 후 값을 반환한다. </br>

```python
car = {"brand": "Ford", "model": "Mustang", "year": 1964}
print(car.setdefault("color", "white"))
print(car.setdefault("bland"))
print(car)
```
```python
# 결과
white
Ford
{'brand': 'Ford', 'model': 'Mustang', 'year': 1964, 'color': 'white'}
```

## ✅ dictionary.update(iterable) -> None
딕셔너리에 지정된 요소(iterable)를 삽입한다. 

📎 iterable	: 딕셔너리 또는 딕셔너리에 삽입될 수 있는 키-값의 쌍으로 이루어진 iterable object
</br> 

```python
car = {"brand": "Ford", "model": "Mustang", "year": 1964}
cat = {"cat": "garby", "age": 12}
cat2 = {"cat": "냐옹이", "age": 12}
tiger = [["tiger", "호랑이"]]

car.update(cat)
car.update(cat2)
car.update(tiger)
print(car)
```
```python
# 결과
{'brand': 'Ford', 'model': 'Mustang', 'year': 1964, 'cat': '냐옹이', 'age': 12, 'tiger': '호랑이'}
```

## ✅ dictionary.values() -> dict_values()
view object를 반환한다. view object는 딕셔너리의 모든 값을 리스트의 형태로 포함한다.

📎 view object는 해당 딕셔너리의 모든 변경 사항을 반영한다.
</br>

```python
car = {"brand": "Ford", "model": "Mustang", "year": 1964}
print(car.values())
car["value"] = 2018
print(car.values())
```
```python
# 결과
dict_values(['Ford', 'Mustang', 1964])
dict_values(['Ford', 'Mustang', 1964, 2018])
```

## References
- <https://www.w3schools.com/python/python_ref_dictionary.asp>

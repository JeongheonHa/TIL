# String(문자열) 함수

📎 `함수 -> 타입`은 반환 타입을 적어 놓은 것입니다.

## ✅ count() -> int
문자열 중 해당 문자의 개수를 반환

```python
>>> a = "hobby"
>>> print(a.count('b')) # 2
```

## ✅ find() -> int
문자열 중 해당 문자가 처음으로 나온 위치를 반환, 만약 찾는 문자나 문자열이 존재하지 않는다면 `-1`을 반환한다.

```python
>>> a = "Python is the best choice"
>>> print(a.find('b'))     # 14
>>> print(a.find('k'))     # -1
```
📎 index는 0부터 시작한다.

## ✅ index() -> int
find와 기능은 동일, 다만 find 함수와 다른 점은 문자열 안에 존재하지 않는 문자를 찾으면 `오류`가 발생한다는 점이다.

```python
>>> a = "Life is too short"
>>> print(a.index('t'))    # 8
>>> print(a.index('k'))

# error
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
ValueError: substring not found
```

## ✅ join() -> str
이전에 정리해 둔게 있어서 아래 링크를 통해 보시면 됩니다.

<https://jh2476.tistory.com/13?category=1089183>

## ✅ upper() -> str
소문자를 대문자로 바꾸어 새로운 문자열 반환. (원본은 바뀌지 않는다.)

```python
>>> a = "hi"
>>> print(a.upper())   # 'HI'
```

## ✅ isupper() -> bool
해당 문자열의 문자가 모두 대문자 인지 확인

```python
>>> a = "My name is Haribo"
>>> print(a.isupper())  # False
```

## ✅ lower() -> str
대문자를 소문자로 바꾸어 새로운 문자열 반환. (원본은 바뀌지 않는다.)

```python
>>> a = "HI"
>>> print(a.lower())   # 'hi'
```
## ✅ islower() -> bool
해당 문자 열의 문자가 모두 소문자 인지 확인

```python
>>> a = "hi"
>>> print(a.islower())  # True
```
## ✅ lstrip() -> str
문자열 중 가장 `왼쪽`에 있는 한 칸 이상의 연속된 `공백`들을 모두 지운 문자열 반환

```python
>>> a = " hi "
>>> print(a.lstrip())
'hi '
```

## ✅ rstrip() -> str
문자열 중 가장 `오른쪽`에 있는 한 칸 이상의 연속된 `공백`을 모두 지운 문자열 반환

```python
>>> a= " hi "
>>> print(a.rstrip())   # ' hi'
```

## ✅ strip() -> str
문자열 `양쪽`에 있는 한 칸 이상의 연속된 `공백`을 모두 지운 문자열 반환

```python
>>> a = " hi "
>>> print(a.strip())    # 'hi'
```

## ✅ replace() -> str
replace(바뀌게 될 문자열, 바꿀 문자열)처럼 사용해서 문자열 안의 특정한 값을 다른 값으로 `치환`한 문자열 반환

```python
>>> a = "Life is too short"
>>> a.replace("Life", "Your leg")  

# 결과
'Your leg is too short'
```


## ✅ split() -> list
이전에 정리해 둔게 있어서 아래 링크를 통해 보시면 됩니다.

<https://jh2476.tistory.com/12?category=1089183>

## ✅ capitialize() -> str
문자열의 첫 문자를 대문자로 만들고 나머지 문자를 모두 소문자로 변환 한 문자열 반환

```python
>>> a = "Hello my name is Haribo"
>>> print(a.capitalize())

# 결과
Hello my name is haribo
```

## ✅ isalpha() -> bool
문자열의 모든 문자가 알파벳, 한글로 되어있는지 확인 (📎 whitespace가 있으면 False)

```python
>>> a = "Hellomynameis하리보"
>>> print(a.isalpha())  # True

>>> a = "Hello myname is Haribo"
>>> print(a.isalpha())  # False
```

## ✅ isalnum() -> bool
문자열의 모든 문자가 알파벳, 한글, 숫자로만 되어있는지 확인 (📎 whilespace가 있으면 False)

```python
>>> a = "HellomynameisHaribo111"
>>> print(a.isalnum())  # True

>>> a = "Hello myname is Haribo111"
>>> print(a.isalnum())  # False
```
## ✅ isdigit() -> bool
문자열의 모든 문자가 숫자로만 되어있는지 확인 (📎 whilespace가 있으면 False)

```python
>>> a = "1234567890"
>>> print(a.isdigit())  # True

>>> a = "1 2 3 4 5 6 7 8 9 0"
>>> print(a.isdigit())  # False
```
## ✅ isdecimal() -> bool
문자열의 모든 문자가 숫자로만 되어있는지 확인  (📎 whilespace가 있으면 False)

```python
>>> a = "11111111"
>>> print(a.isdecimal())    # True

>>> a = "1 1 1 1 1 1 1 1"
>>> print(a.isdecimal())    # False
```

## ✅ isspace() -> bool
문자열의 모든 문자가 whitespace인지 확인 

```python
>>> a = "\t\f\v "
>>> print(a.isspace())  # True
```

## ✅ swapcase() -> str
문자열의 모든 문자를 대문자 -> 소문자, 소문자 -> 대문자로 변환한 문자열 반환

```python
a = "Hello My Name is Haribo"
print(a.swapcase())     # hELLO mY nAME IS hARIBO
```

## ✅ title() -> str
whitespace를 기준으로 각 단어의 첫 문자를 대문자로 나머지는 소문자로 변환한 문자열 반환

```python
>>> a = "hELLO mY nAME iS hARIBO"
>>> print(a.title())    # Hello My Name Is Haribo
```
```python
>>> a = "hEL\tLO mY\fnAME IS hAR\vIBO"
>>> print(a.title())

# 결과
Hel     Lo My
             Name Is Har
                        Ibo
```

## ✅ istitle() -> bool
whitespace를 기준으로 각 단어의 첫 문자가 대문자로 나머지 문자가 소문자로 되어있는지 확인

```python
>>> a = "Hello My Name Is Haribo"
>>> print(a.istitle())  # True

>>> a = "HE\tLLO MY NAME IS HARIBO"
>>> print(a.istitle())  # False
```

## ✅ startswith() -> bool
문자열의 첫번째 단어가 해당 문자열로 이루어져있는지 확인 (📎 대소문자 구분 해야한다.)

```python
>>> a = "HE\tLLO MY NAME IS HARIBO"
>>> print(a.startswith("HE"))   # True

>>> a = "HE\tLLO MY NAME IS HARIBO"
>>> print(a.startswith("he"))   # False
```

## ✅ endswith() -> bool
문자열의 마지막 단어가 해당 문자열로 이루어져있는지 확인 (📎 대소문자 구분 해야한다.)

```python
>>> a = "HE\tLLO MY NAME IS HARIBO"
>>> print(a.endswith("HARIBO")) # True

>>> a = "HE\tLLO MY NAME IS HARIBO"
>>> print(a.endswith("haribo")) # False
```

# References
- <https://wikidocs.net/13>
- <https://www.w3schools.com/python/python_ref_string.asp>

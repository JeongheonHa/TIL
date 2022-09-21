# asterisk(*)

📌 파이썬에서의 asterisk(*)기호가 갖는 기능에 대해서 살펴보자.

## ✅ 곱셈과 거듭제곱
설명 대신 코드를 보면 이해하기 쉽다.
</br>

```python
print(2*3)  # 6, 곱셈
print(2**3) # 8, 거듭제곱
```

## ✅ 리스트의 확장
*를 사용하여 리스트를 간단히 확장할 수 있다.
</br>

```python
list = [1]*5
print("list {}".format(list))   # list [1, 1, 1, 1, 1]
```
```python
tuple = (1,)*5
print("tuple {}".format(tuple)) # tuple (1, 1, 1, 1, 1)

# 주의
tuple = (1)*5
print("tuple {}".format(tuple)) # tuple 5
```

## ✅ 가변 인자
`*arg`를 매개변수로 지정하면 함수를 정의할 때 매개변수로 여러개의 인자를 받아 올 수 있는 기능을 제공한다.   

</br>

```python
def my_function(*kids):
  print("The youngest child is " + kids[2]) # The youngest child is Linus

my_function("Emil", "Tobias", "Linus") 
```
</br>

`**kwargs`를 매개변수로 지정하면 함수를 정의할 때 매개변수로 딕셔너리를 받아 여러 요소에 접근할 수 있는 기능을 제공한다.
</br>

```python
def my_function(**kid):
  print("His last name is " + kid["lname"]) # His last name is Refsnes

my_function(fname = "Tobias", lname = "Refsnes")
```

## ✅ Unpacking
리스트나 튜플을 출력할 때 `[1, 2, 3, 4, 5]`형식으로 출력되는 것을 `1 2 3 4 5`형식으로 출력시킬 수 있는 기능을 제공한다.   

📎 이렇게 하면 for문을 이용해서 리스트의 내용을 출력하지 않아도 되므로 코드가 훨씬 간결해진다.
</br>

```python
# list unpacking
test = [1, 2, 3, 4]
print(*test) # 1 2 3 4

# tuple unpacking
test = (5, 6, 7, 8)
print(*test) # 5 6 7 8
```

</br>
각 변수 중 하나에 가변적으로 할당하고 싶은 경우 다음과 같이 사용할 수 있다.
</br>

```python
test = [1, 2, 3, 4, 5]

*a, b = test
print(a, b) # [1, 2, 3, 4], 5

a, *b, c = test
print(a, b, c) # 1, [2, 3, 4], 5
```


# References
- <https://www.w3schools.com/python/gloss_python_function_arbitrary_arguments.asp>
- <https://hwiyong.tistory.com/193>

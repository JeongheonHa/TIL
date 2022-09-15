# sort vs sorted

- `sort()` : `리스트명.sort( )` 형식으로 "리스트형의 메소드"​​이며 리스트 원본값을 직접 수정한다.

- `sorted()` : `sorted( iterable )` 형식으로 "내장 함수"이며 리스트 원본 값은 그대로이고 정렬 값을 반환한다. `()`안에는 반복이 가능한 iterable 자료형을 넣는다.

📎 오름차순은 사전 순을 말하며 영어는 abcd..., 한글은 가나다..., 숫자는 1234... 순서를 의미하며 정렬 함수의 기본값이다.

두 개의 차이를 알아보았으니 실습을 통해 확인을 해보자.

## 1. sort()

```python
>>> a1 = [6, 3, 9]
>>> print('a1:', a1)
>>> a2 = a1.sort() # 원본을 정렬하고 수정합니다(in-place)
>>> print('-----정렬 후-----')
>>> print('a1:', a1)
>>> print('a2:', a2)

# 결과
a1: [6, 3, 9]
-----정렬 후-----
a1: [3, 6, 9]
a2: None
```

📎 sort()의 반환 값은 None이라는 것을 주의하자.

## 2. sorted()

```python
>>> b1 = [6, 3, 9]
>>> print('b1:', b1)
>>> b2 = sorted(b1) # 원본은 유지하고 정렬한 새 리스트를 만듭니다
>>> print('-----정렬 후-----')
>>> print('b1:', b1)
>>> print('b2:', b2)

# 결과
 b1: [6, 3, 9]
-----정렬 후-----
b1: [6, 3, 9]
b2: [3, 6, 9]
```

## 3. key, reverse 매개변수

sort함수와 sorted함수는 공통적으로 정렬 기준을 정하는 매개변수인 `key`와 `reverse`를 가지고 있다.

### reverse   
기본값은 reverse=False(오름차순)로 정의되어 있으며   
reverse=True를 매개변수로 입력하면 내림차순으로 정렬할 수 있다.

```python
>>> num_list = [15, 22, 8, 79, 10]
>>> num_list.sort(reverse=True)
>>> print(num_list)
[79, 22, 15, 10, 8]

>>> print(sorted(['좋은하루','good_morning','굿모닝','niceday'], reverse=True))
['좋은하루', '굿모닝', 'niceday', 'good_morning']
```
 
### key   
key 매개변수에는 정렬을 목적으로 하는 함수를 값으로 넣으며 lambda를 이용할 수 있다.   
key 값을 기준으로 정렬되고 기본값은 오름차순이다. 

```python
>>> str_list = ['좋은하루','good_morning','굿모닝','niceday']
>>> print(sorted(str_list, key=len))  # 함수
['굿모닝', '좋은하루', 'niceday', 'good_morning']

>>> print(sorted(str_list, key=lambda x : x[1]))  # 람다
['niceday', 'good_morning', '굿모닝', '좋은하루']
```

tuple을 이용하여 정렬 기준을 정할 수 도 있다.

```python
>>> tuple_list = [('좋은하루', 0),
    	          ('niceday', 1), 
    	          ('좋은하루', 5), 
    	          ('good_morning', 3), 
    	          ('niceday',5)]
                  
>>> tuple_list.sort(key=lambda x : (x[0], x[1]))  # '-'부호를 이용해서 역순으로 가능
>>> print(tuple_list)
[('good_morning', 3), ('niceday', 1), ('niceday', 5), ('좋은하루', 0), ('좋은하루', 5)]
```
# References
- <https://ooyoung.tistory.com/59>   
- <https://blog.naver.com/PostView.nhn?isHttpsRedirect=true&blogId=wideeyed&logNo=221745416992&redirect=Dlog&widgetTypeCall=true&directAccess=false>

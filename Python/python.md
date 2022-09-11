## 1. Type Hint
- 변수의 타입을 지정해주는 것
- 가독성이 좋아지며 버그 발생 확률을 줄인다.
- 동적으로 할당될 수 있으므로 주의가 필요하다.
```python
def fn(a: int) -> bool:
```

## 2. List Comprehension
- 기존 리스트를 기반으로 새로운 리스트를 만들어내는 구문
- 대체로 표현식은 2개를 넘지 않아야한다.
- 역할별로 줄 구분을 하면 가독성이 높아진다.
```python
[n * 2 for n in range(1, 10 + 1) if n % 2 == 1]
```

## 3. Generator
- 루프의 반복 동작을 제어할 수 있는 루틴 형태
- 제너레이터만 생성해두고 필요할 때 언제든 숫자를 만들어낼 수 있다.
- `yield` 구문을 사용하면 제너레이터 리턴
- 다음 값을 생성하려면 `next()`로 추출
- 여러 타입의 값을 하나의 함수에서 생성하는 것도 가능
```python
def generator():
    yield 1
    yield 'string'
    yield True

g = generator()
next(g) # 1
next(g) # 'string'
next(g) # True
```

## 4. range
- 제너레이터 역할을 하는 range클래스를 리턴
- 생성 조건만 정해두고 나중에 필요할 때 생성
```python
list(range(5))          # [0, 1, 2, 3, 4]
range(5)                # range(0, 5)
type(range(5))          # <class 'range'>
for i in range(5):
    print(i, end=' ')   # 0 1 2 3 4
```

## 5. enumerate
- 여러가지 자료형을 인텍스를 포함한 enumerate 객체로 리턴
```python
a = [1,2,3,2,45,2,5]
list(enumerate(a))  # [(0, 1), (1, 2), (2, 3), (4, 45), (5, 2), (6, 5)]

# for 활용
a = ['a1', 'b2', 'c3']

for i, v in enumerate(a):
    print(i, v)
```

## 6. // 연산자
- `몫`을 구하는 연산자
- `몫`과 `나머지`를 한 번에 구할 경우
```python
divmod(5, 3)    # (1, 2)
```

## 7. print
```python
print('A1', 'B2')   # A1 B2, 줄바꿈 적용되어 있음
print('A1', 'B2', sep=',')    # A1,B2 (구분자를 지정)
print('aa', end=' ')    # aa (줄바꿈 대신 끝을 공백으로 대체)
print('bb') # aa bb
```
- 리스트를 출력할 때 `join()`으로 묶어서 처리한다.
```python
a = ['A', 'B']
print(' '.join(a))  # A B
```
- 변수 출력
```python
idx = 1
fruit = "Apple"

print('{0}: {1}'.format(idx + 1, fruit))    # 2: Apple
print('{}: {}'.format(idx + 1, fruit))      # 2: Apple
print(f'{idx + 1}: {fruit}')                # 2: Apple (파이썬 3.6+ 밑에 버전은 동작하지 않는다.)
```

## 8. pass
- 널 연산으로 아무것도 하지 않는 기능을 가진다.
- 목업 인터페이스부터 구현한 다음에 추후 구현을 진행할 수 있게한다.
```python
class MyClass(object):
    def method_a(self):
        pass

    def method_b(self):
        print("Method b")

c = MyClass()
```

## 9. locals
- 로컬 심볼 테이블 딕셔너리를 가져오는 메서드
- 클래스 메서드 내부의 모든 로컬 변수를 출력해 주기 때문에 디버깅에 유용하다.
```python
import pprint   # pprint를 이용하면 보기 좋게 줄바꿈 처리를 해준다.
pprint.pprint(locals())
```

## 10. 코딩 스타일
- 함수의 기본 값으로 `가변 객체`를 사용하지 않는다. (함수가 객체를 수정하면 기본값이 변경되기 때문에)
```python
def foo(a, b = []):             # No
def foo(a, b: Mapping = {}):    # No
```

- 함수의 기본 값으로 `불변 객체`를 사용한다.
```python
def foo(a, b=None):                         # Yes
    if b is None:
        b = []

def foo(a, b: Optional[Sequence] = None):   # Yes
    if b is None:
        b = []
```

- True, False를 판별할 때는 암시적인 방법을 사용한다.
```python
if not users:       # Yes

if len(users) == 0: # No
```

# Reference
> 파이썬 알고리즘 인터뷰 - 박상길

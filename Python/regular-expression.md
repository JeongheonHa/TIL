# 정규 표현식
정규 표현식은 복잡한 문자열을 처리할 때 사용하는 기법이다.

## 1. 메타 문자

먼저, 메타 문자란 원래 그 문자가 가진 뜻이 아닌 특별한 용도로 사용하는 문자를 말하며 메타 문자에는 다음과 같은 것들이 있다.
```python
. ^ $ * + ? { } [ ] \ | ( )
```

### 문자 클래스 `[]`   
- `[]` 사이의 문자들과 매치라는 의미이다.

`[]`사이에 하이픈(`-`)을 사용하면 두 문자 사이의 범위를 의미한다.

예를 들자면,  `[abc]`은 "a, b, c 중 한 개의 문자와 매치"를 의미한다. 반면, `abc`는 "abc로된 문자와 매치"를 의미하며 혼동 하지 말자.

```python
# 예시
[a-zA-Z] : 알파벳 모두
[0-9] : 숫자
```

다음은 자주 사용되는 정규식을 별도의 표기법으로 정의한것이다.

- `\d` - `숫자`와 매치, `[0-9]`와 동일한 표현식이다.
- `\D` - 숫자가 아닌 것과 매치, `[^0-9]`와 동일한 표현식이다.
- `\s` - `whitespace` 문자와 매치, `[ \t\n\r\f\v]`와 동일한 표현식이다. 맨 앞의 빈 칸은 공백문자(space)를 의미한다.
- `\S` - whitespace 문자가 아닌 것과 매치, `[^ \t\n\r\f\v]`와 동일한 표현식이다.
- `\w` - `문자+숫자`(alphanumeric)와 매치, `[a-zA-Z0-9_]`와 동일한 표현식이다.
- `\W` - 문자+숫자(alphanumeric)가 아닌 문자와 매치, `[^a-zA-Z0-9_]`와 동일한 표현식이다.


> `주의` : `[]`안에 사용되는 `^`는 반대의 의미하기 때문에 []안에 쓰이는지 밖에 쓰이는지 주의하면서 사용하자.

### Dot(`.`)
- Dot(`.`)은 `\n`을 제외한 모든 문자와 매치됨을 의미한다.
```python
a.b     # a + 모든 문자 + b
```
```python
a[.]b   # a + Dot(.)문자 + b
```

### 반복(`*`)
- 앞의 문자가 무한대로 반복될 수 있다는 것을 의미한다.

```python
ca*t    # caaaaaat와 매치
```
> a가 전현 없이 반복이 `0번`인 경우도 매치된다는 점을 주의하자.

### 반복(`+`)
- `*`와 기능은 같으며 단지 반복이 `1번` 이상이 경우부터 매치가 된다는 점이다.

### 반복(`{m,n,?}`)
- 반복 횟수를 제한하는데 사용된다.

1. {m}   
지정한 반복을 가진 문자열만 매치된다.
```python
ca{2}t      # c + a(반드시 2번 반복) + t
```

2. {m,n}   
해당 범위 내의 반복은 모두 매치된다. (m<= x <= n)
```python
ca{2,5}t    # c + a(2~5회 반복) + t
```

3. ?   
{0,1}과 같다.
```python
ab?c        # a + b(있어도 되고 없어도 된다) + c
```

## 2. re 모듈
파이썬은 정규 표현식을 지원하기 위해 `re`(regular expression)모듈을 제공한다.
```python
import re
p = re.complie('ab*')
```
re.compile을 사용하여 정규 표현식을 컴파일하고 결과로 받은 객체 p(컴파일된 패턴 객체)를 사용하여 작업한다.
> 여기서 패턴이란 정규식을 컴파일한 결과이다.

### 문자열 검색
컴파일된 패턴 객체는 다음과 같은 4개의 메서드를 제공한다.
- `match()` : 문자열의 처음부터 정규식과 매치되는지 조사
- `search()` : 문자열 전체를 검색하여 정규식과 매치되는지 조사
- `findall()` : 정규식과 매치되는 모든 문자열을 리스트로 반환
- `finditer()` : 정규식과 매치되는 모든 문자열을 반복 가능한 객체로 반환   

   
match,search는 정규식과 매치가 되면 `match객체`를 반환하고 매치가 안되면 `NONE`을 반환한다.

1. match()
```python
# 컴파일된 패턴 객체 생성
>>> import re
>>> p = re.compile('[a-z]+')
```
```python
>>> m = p.match("python")
>>> print(m)

# match 객체
<re.Match object; span=(0, 6), match='python'>
```
```python
>>> m = p.match("3 python")
>>> print(m)
None
```

2. search()
```python
>>> m = p.search("python")
>>> print(m)
<re.Match object; span=(0, 6), match='python'>
```
```python
>>> m = p.search("3 python")
>>> print(m)
<re.Match object; span=(2, 8), match='python'>
```

3. findall()
```python
>>> result = p.findall("life is too short")
>>> print(result)
['life', 'is', 'too', 'short']
```

4. finditer()
```python
>>> result = p.finditer("life is too short")
>>> print(result)
<callable_iterator object at 0x01F5E390>
>>> for r in result: print(r)
...
<re.Match object; span=(0, 4), match='life'>
<re.Match object; span=(5, 7), match='is'>
<re.Match object; span=(8, 11), match='too'>
<re.Match object; span=(12, 17), match='short'>
```
반복 가능한 객체가 포함하는 각각의 요소는 match객체이다.

### match 객체의 메서드
- `group()` : 매치된 문자열을 반환
- `start()` : 매치된 문자열의 시작 위치 반환
- `end()` : 매치된 문자열의 끝 위치 반환
- `span()` : 매치된 문자열의 (start, end)에 해당하는 tuple을 반환

```python
>>> m = p.search("3 python")
>>> m.group()
'python'
>>> m.start()
2
>>> m.end()
8
>>> m.span()
(2, 8)
```

### 모듈 단위로 수행하기
```python
# 기존의 re.compile을 이용하는 방법
>>> p = re.compile('[a-z]+')
>>> m = p.match("python")

# 축약된 형태
>>> m = re.match('[a-z]+', "python")
```
한번 만든 패턴을 여러 번 사용할 때는 기존의 방법을 사용하고 한 번만 사용하고 말거면 축약된 형태를 사용하는 것이 더 좋다.

## 3. 컴파일 옵션
- `DOTALL`(S) : `.`이 줄바꿈 문자(`\n`)를 포함하여 모든 문자와 매치할 수 있도록 한다.
- `IGNORECASE`(I) : 대소문자에 관계없이 매치할 수 있도록 한다.
- `MULTILINE`(M) : 여러 줄과 매치할 수 있도록 한다. (`^`, `$` 메타문자의 사용과 관계가 있는 옵션이다.)
- `VERBOSE`(X) : verbase모드를 사용할 수 있도록 한다. (복잡한 정규식을 보기 쉽게 만들 수 있다.)

옵션을 사용할 때는 `re.DOTAL`L 또는 `re.S`처럼 사용한다.

1. DOTALL(S)
```python
>>> p = re.compile('a.b', re.DOTALL)
>>> m = p.match('a\nb')
>>> print(m)
<re.Match object; span=(0, 3), match='a\nb'>
```

2. IGNORECASE(I)
```python
>>> p = re.compile('[a-z]+', re.I)
>>> p.match('python')
<re.Match object; span=(0, 6), match='python'>
>>> p.match('Python')
<re.Match object; span=(0, 6), match='Python'>
>>> p.match('PYTHON')
<re.Match object; span=(0, 6), match='PYTHON'>
```

3. MULTILINE(M)   
`^`는 문장열의 처음을 의미하고, `$`는 문자열의 마지막을 의미한다. 예를 들어 `^python`은 문자열의 처음은 항상 python으로 시작해야 매치되고, `python$`이라면 문자열의 마지막은 항상 python으로 끝나야 매치된다는 의미이다.

```python
# ^
import re
p = re.compile("^python\s\w+")

data = """python one
life is too short
python two
you need python
python three"""

print(p.findall(data))

# 결과
['python one']
```

```python
# re.MULTILINE
import re
p = re.compile("^python\s\w+", re.MULTILINE)

data = """python one
life is too short
python two
you need python
python three"""

print(p.findall(data))

# 결과
['python one', 'python two', 'python three']
```

4. VERBOSE(X)   

```python
charref = re.compile(r'&[#](0[0-7]+|[0-9]+|x[0-9a-fA-F]+);')
```
```python
charref = re.compile(r"""
 &[#]                # Start of a numeric entity reference
 (
     0[0-7]+         # Octal form
   | [0-9]+          # Decimal form
   | x[0-9a-fA-F]+   # Hexadecimal form
 )
 ;                   # Trailing semicolon
""", re.VERBOSE)
```
re.VERBOSE 옵션을 사용하면 문자열에 사용된 whitespace는 컴파일할 때 제거된다(단 [ ] 안에 사용한 whitespace는 제외). 그리고 줄 단위로 #기호를 사용하여 주석문을 작성할 수 있다.

## 4. 백슬래시
"\section"문자열을 찾기 위한 정규식을 만든 때 아래와 같이 사용한다면
```python
\section
```
이 정규식은 `\s`로 해석되어 의도치 않은 정규식이 적용된다.   
이에 escape 처리를 해서 `\\`로 `\`하나를 표현해야 하지만  파이썬 리터럴 정규식 엔진에는 파이썬 뭄ㄴ자열 리터럴 규칙에 따라 `\\`이 `\`로 변경된다.   
결국 `\\`을 전달하기 위해서는 아래와 같이`\\\\`4번을 사용해야한다.
```python
>>> p = re.compile('\\\\section')
```
하지만, 이것은 복잡해보이기에 Raw String이라는 파이썬 문법이 만들어 졌다.
```python
>>> p = re.compile(r'\\section')
```
위와 같이 정규식 문자열 앞에 r 문자를 삽입하면 이 정규식은 Raw String 규칙에 의하여 백슬래시 2개 대신 1개만 써도 2개를 쓴 것과 동일한 의미를 갖게 된다.

## 5. 열소비가 없는 메타 문자
지금 까지 본 메타 문자는 매치가 진행될 때 현재 매치되고 있는 문자열의 위치가 변경(소비)되지만 문자열을 소비시키지 않는 메타 문자도 존재한다.

### |
or 과 동일한 의미이다.
```python
>>> p = re.compile('Crow|Servo')
>>> m = p.match('CrowHello')
>>> print(m)
<re.Match object; span=(0, 4), match='Crow'>
```

### ^
문자열의 맨 처음과 일치함을 의미한다.
```python
>>> print(re.search('^Life', 'Life is too short'))
<re.Match object; span=(0, 4), match='Life'>
>>> print(re.search('^Life', 'My Life'))
None
```

### $
문자열의 맨 마지막과 일치함을 의미한다.
```python
>>> print(re.search('short$', 'Life is too short'))
<re.Match object; span=(12, 17), match='short'>
>>> print(re.search('short$', 'Life is too short, you need python'))
None
```
> `^`, `$`를 문자 그 자체로 매치하고 싶은 경우 `\^`, `\$`를 사용한다.

### \A
`^`과 동일한 기능이며 단지`\A`는 줄과 상관없이 전체 문자열의 처음하고만 매치된다.

### \Z
$과 동일한 기능이며 단지 `\Z`는 줄과 상관없이 전체 문자열의 끝하고만 매치된다.

### \b
단어 구분자이며 `공백`을 구분할 때 사용된다.
```python
>>> p = re.compile(r'\bclass\b')
>>> print(p.search('no class at all'))  
<re.Match object; span=(3, 8), match='class'>
```
`\b`는 파이썬 리터럴 규칙에 의해 백스페이스를 의미하므로 단어 구분자임을 명시하기 위해 `r'\bclass\b`처럼 Raw string임을 얄려주는 기호 `r`을 반드시 붙여준다.

### \B
`\b`와 반대의 의미이다. 즉 공백으로 구분된 단어가 아닌 경우만 매치된다.
```python
>>> p = re.compile(r'\Bclass\B')
>>> print(p.search('no class at all'))  
None
>>> print(p.search('the declassified algorithm'))
<re.Match object; span=(6, 11), match='class'>
>>> print(p.search('one subclass is'))  # class 앞뒤에 공백이 하나라도 있으면 매치가 안된다.
None
```

### Grouping `()`
1. 문자열이 반복되는지 조사하고 싶을 때 사용한다.
```python
>>> p = re.compile('(ABC)+')
>>> m = p.search('ABCABCABC OK?')
>>> print(m)
<re.Match object; span=(0, 9), match='ABCABCABC'>
>>> print(m.group())
ABCABCABC
```

2. 특정 정규식에 매치되는 문자열만 뽑아내고 싶을 때 사용한다.
```python
>>> p = re.compile(r"(\w+)\s+\d+[-]\d+[-]\d+")
>>> m = p.search("park 010-1234-1234")
>>> print(m.group(1))
park
```
`group(index)`를 통해 `\w+` 정규식에 매치되는 이름 문자열을 뽑아낼 수 있다.
> index는 0부터 시작이다.

3. Grouping된 문자열을 재참조할 수 있다.   
`\숫자`는 정규식의 그룹 중 숫자에 해당하는 그룹을 가리킨다.
```python
>>> p = re.compile(r'(\b\w+)\s+\1')
>>> p.search('Paris in the the spring').group()
'the the'
```
`(\b\w+)\s+\1`은 `(그룹) + " " + 그룹과 동일한 단어`와 매치됨을 의미하며 두 개의 동일한 단어를 연속적으로 사용해야만 매치된다. 

4. Grouping된 문자열에 이름을 붙일 수 있다.   
group을 `index`가 아닌 그룹의 `이름`으로 참조할 수 있게 해준다.
```python
(\w+) --> (?P<name>\w+)   # (?P<그룹명>...)
```
```python
>>> p = re.compile(r"(?P<name>\w+)\s+((\d+)[-]\d+[-]\d+)")
>>> m = p.search("park 010-1234-1234")
>>> print(m.group("name"))
park
```
5. 그룹 이름으로 재참조도 가능하다.
```python
>>> p = re.compile(r'(?P<word>\b\w+)\s+(?P=word)')
>>> p.search('Paris in the the spring').group()
'the the'
```

## 6. 전방 탐색
전방 탐색에는 **긍정형 전방 탐색**과 **부정형 전방 탐색**이 있으며 예를 보며 알아보자.

1. 긍정형 전방 탐색 (`(?=...)`)   
`...`에 해당되는 정규식과 매치되어야 하며 조건이 통과되어도 문자열이 소비되지 않는다.
```python
>>> p = re.compile(".+(?=:)")
>>> m = p.search("http://google.com")
>>> print(m.group())
http
```
기존 정규식과 검색에서는 동일한 효과를 발휘하지만 `:` 에 해당하는 문자열이 정규식 엔진에 의해 소비되지 않아(검색에는 포함되지만 검색 결과에는 제외됨) 검색 결과에서는 `:`이 제거된 후 돌려주는 효과가 있다.

2. 부정형 전방 탐색 (`(?!...)`)   
`...`에 해당되는 정규식은 문자열이 있는지 조사하는 과정에서 문자열이 소비되지 않으므로 `...`가 아니라고 판단되면 그 이후 정규식 매치가 진행된다.
```python
# 파일명 + . + bat확장자를 제외하고 매치하는 방법
.*[.]([^b].?.?|.[^a]?.?|..?[^t]?)$

# 부정형 전방 탐색을 사용한 경우
.*[.](?!bat$).*$

# exe 조건을 추가한 경우
.*[.](?!bat$|exe$).*$
```

## 7. 문자열 바꾸기
`sub`메서드를 사용하면 정규식과 매치되는 부분을 다른 문자로 쉽게 바꿀 수 있다.
```python
sub(바꿀 문자열, 바꿀 대상 문자열)
```
```python
>>> p = re.compile('(blue|white|red)')
>>> p.sub('colour', 'blue socks and red shoes')
'colour socks and colour shoes'

# 딱 한 번만 바꾸고 싶은 경우
>>> p.sub('colour', 'blue socks and red shoes', count=1)  # count를 이용해 바꾸는 횟수를 제한할 수 있다.
'colour socks and red shoes'
```

### subn 메서드
sub메서드와 동일한 기능을 하지만 반환 결과를 `tuple`로 돌려준다.
```python
>>> p = re.compile('(blue|white|red)')
>>> p.subn( 'colour', 'blue socks and red shoes')
('colour socks and colour shoes', 2)  # (변경된 문자열, 바꾼 횟수)
```

### sub 메서드 사용시 참조 구문 사용하기
`\g<그룹이름>`을 사용하면 정규식의 그룹 이름을 참조할 수 있다.
```python
# 이름 + 전화번호 -> 전화번호 + 이름
>>> p = re.compile(r"(?P<name>\w+)\s+(?P<phone>(\d+)[-]\d+[-]\d+)")
>>> print(p.sub("\g<phone> \g<name>", "park 010-1234-1234"))
010-1234-1234 park

# 그룹 이름 대신 참조 변수를 사용하는 방법
>>> p = re.compile(r"(?P<name>\w+)\s+(?P<phone>(\d+)[-]\d+[-]\d+)")
>>> print(p.sub("\g<2> \g<1>", "park 010-1234-1234"))
010-1234-1234 park
```

### sub 메서드의 매개변수로 함수 넣기
```python
>>> def hexrepl(match):
...     value = int(match.group())
...     return hex(value)
...
>>> p = re.compile(r'\d+')
>>> p.sub(hexrepl, 'Call 65490 for printing, 49152 for user code.')
'Call 0xffd2 for printing, 0xc000 for user code.'
```
sub의 첫번 째 매개변수로 함수를 사용할 경우 해당 함수의 첫 번째 매개변수에는 정규식과 매치된 match객체가 입력되고 매치되는 문자열은 함수의 반환 값으로 바뀌게 된다.

## 8. Greedy vs Non-Greedy
Greedy는 탐욕스러움을 뜻하는데 정규식에서 Greedy는 어떤 의미인지 알아보자.

### Greedy
```python
>>> s = '<html><head><title>Title</title>'
>>> len(s)
32
>>> print(re.match('<.*>', s).span())
(0, 32)
>>> print(re.match('<.*>', s).group())
<html><head><title>Title</title>

# Non-Greedy 문자 '?'를 사용한 경우
>>> print(re.match('<.*?>', s).group())
<html>
```
위의 예제와 같이 `*`는 탐욕스러워서 `<html>`만이 아닌 전체 문자열을 소비 해버렸다. 이럴 경우 Non-Geedy문자 `?`를 이용해서 `*`의 탐욕을 제한할 수 있다.   

Non-Greedy 문자인 `?`는 `*?`, `+?`, `??`, `{m,n}?`와 같이 사용할 수 있다.

# References
> 점프 투 파이썬

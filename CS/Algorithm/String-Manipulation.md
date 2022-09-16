# 문자열 조작

## 1. Valid Palindrome

팰린드롬 : 뒤집어도 같은 말이 되는 단어 또는 문장을 말한다.

Q. 주어진 문자열이 팰린드롬인지 확인하라. 대소문자를 구분하지 않으며, 영문자와 숫자만 대상으로 한다.   

Example 1:

```python
Input: s = "A man, a plan, a canal: Panama"
Output: true
```
Example 2:

```python
Input: s = "race a car"
Output: false
```

Solution:
```python
import re

class Solution:
    def isPalindrome(self, s: str):
        # 문자열의 문자들을 모두 소문자로 만든다.
        s.lower()
        # 정규식을 이용해 숫자와 알파벳을 제외하고 모두 s 문자열에서 제거
        s = re.sub('[^0-9a-z]', '', s)
        # 슬라이싱을 이용해 문자열을 reverse해서 각각의 문자들을 비교
        return s == s[::-1]
```

- sub(정규식, 치환 문자, 대상 문자열)
- `[::-1]` : 문자열 reverse

📎 대부분의 문자열 작업은 슬라이싱으로 처리하는 것이 가장 빠르다.


## 2. Reverse String

Q. 문자열을 뒤집는 함수를 작성하라. 입력값은 문자 배열이며, 리턴 없이 리스트 내부를 직접 조작하라.

Example 1:
```python
Input: s = ["h","e","l","l","o"]
Output: ["o","l","l","e","h"]
```
Example 2:
```python
Input: s = ["H","a","n","n","a","h"]
Output: ["h","a","n","n","a","H"]
```

Solution:
```python
class Solution:
    def reverseString(self, s: List[str]) -> None:
        s.reverse()
```
📎 reverse()는 List에서만 제공된다.

## 3. Reorder Log Files

Q. 로그를 다음과 같은 기준으로 재정렬하라.    
1. 로그의 가장 앞부분은 식별자다.
2. 문자로 구성된 로그가 숫자 로그보다 앞에 온다.
3. 식별자는 순서에 영향을 끼치지 않지만, 문자가 동일한 경우 식별자 순으로 한다.
4. 숫자 로그는 입력 순서대로 한다.

Example 1:

```python
Input: logs = ["dig1 8 1 5 1","let1 art can","dig2 3 6","let2 own kit dig","let3 art zero"]
Output: ["let1 art can","let3 art zero","let2 own kit dig","dig1 8 1 5 1","dig2 3 6"]
```

Example 2:

```python
Input: logs = ["a1 9 2 3 1","g1 act car","zo4 4 7","ab1 off key dog","a8 act zoo"]
Output: ["g1 act car","a8 act zoo","ab1 off key dog","a1 9 2 3 1","zo4 4 7"]
```

Solution:

```python
from typing import List


class Solution:
    def reorderLogFiles(self, logs: List[str]) -> List[str]:
        letters, digits = [], []
        # 리스트의 요소인 문자열을 하나하나 꺼낸다.
        for log in logs:
            # 식별자를 제외한 리스트의 log들이 숫자인지 문자인지 판단
            if log.split()[1].isdigits():
                digits.append(log)
            else:
                letters.append(log)
        # 동일한 문자는 식별자 순으로 정렬
        letters.sort(key = lambda x: (x.split()[1:], x.split()[0]))
        return letters + digits
```

📎 식별자를 제외한 문자열 [1:]을 키로 하여 정렬하며, 동일한 경우 후순위로 식별자 [0]를 지정해 정렬한다.    

즉, key값 2개를 tuple로 묶어서 정의한 것이고 ()안의 순서는 우선순위 대상을 우선순위 순서에 맞춰 정의한 것이다.

## 4. Most Common Word

Q. 금지된 단어를 제외한 가장 흔하게 등장하는 단어를 출력하라. 대소문자 구분을 하지 않으며, 구두점(마침표, 쉼표 등) 또한 무시한다.

Example 1:

```python
Input: paragraph = "Bob hit a ball, the hit BALL flew far after it was hit.", banned = ["hit"]
Output: "ball"
```

Example 2:

```python
Input: paragraph = "a.", banned = []
Output: "a"
```

Solution:

```python
from typing import List
import collections, re


class Solution:
    def mostCommonWord(self, paragraph: str, banned: List[str]) -> str:
        # 금지 문자열을 제외한 모든 문자를 소문자 문자열 리스트로 만든다.
        words = [word for word in re.sub(r'[^\w]', ' ', paragraph).lower().split() if word not in banned]
        # words리스트를 Counter 객체로 변환
        counts = collections.Counter(words)
        # 가장 많이 등장하는 단어를 포함한 요소의 [0][0]값을 출력
        return counts.most_common(1)[0][0]
```
1. 대소문자와 구두점, 금지된 문자열을 무시하고 리스트로 반환하는 전제 조건을 만든다.
2. 조건을 충족하는 리스트를 Counter객체로 만든다.
3. most_common메서드를 이용해 가장 많이 사용된 요소의 첫 번째 요소를 반환한다.

## 5. Group Anagrams

📎 애너그램 : 문자를 재배열하여 다른 뜻을 가진 단어로 바꾸는 것   

Q. 문자열 배열을 받아 애너그램 단위로 그룹핑하라.


Example 1:

```python
Input: strs = ["eat","tea","tan","ate","nat","bat"]
Output: [["bat"],["nat","tan"],["ate","eat","tea"]]
```
Example 2:

```python
Input: strs = [""]
Output: [[""]]
```

Solution:

```python
from typing import List
import collections


class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        # 기본값이 []인 defaultdict객체 생성
        anagrams = collections.defaultdict(list)    # 존재하지 않는 키를 넣을 경우 KeyError 발생하므로 defaultdict으로 생성
        
        for word in strs:
            # key에 따라 word(value)추가
            anagrams[''.join(sorted(word))].append(word)    # 애너그램 관계에 있는 단어를 정렬함으로써 동일한 단어로 만든다.
        # value 값만 리스트로 반환
        return list(anagrams.values())
```
1. 애너그램 관련 문자열을 문자 정렬과 딕셔너리 구조를 이용하여 하나의 key로 만든다.
2. 애너그램 관련 문자열들을 해당 key에 value값으로 넣는다.
3. value값 만 뽑아서 리스트로 반환
## 6. Longest Palindrome Substring

Q. 가장 긴 팰린드롬 부분 문자열을 출력하라.

Example 1:

```python
Input: s = "babad"
Output: "bab"
```
Explanation: "aba" is also a valid answer.

Example 2:

```python
Input: s = "cbbd"
Output: "bb"
```

Solution:

```python
class Solution:
    def longestPalindrome(self, s: str) -> str:
        # 팰린드롬 판별 및 투 포인터 확장
        def expand(left: int, right: int) -> str:
            while left >=0 and right < len(s) and s[left] == s[right]:
                left -= 1
                right += 1
            return s[left + 1:right]
        
        # 해당 사항이 없을 때 빠르게 리턴
        if len(s) < 2 or s == s[::-1]:
            return s
        
        result = ''
        # 슬라이딩 윈도우 우측으로 이동
        for i in range(len(s) -1):
            result = max(result,
                            expand(i, i + 1),   # 짝수일 경우
                            expand(i, i + 2),   # 홀수일 경우
                            key=len)
        return result
```
1. 슬라이딩 윈도우를 이용해 인덱스 0부터 우측으로 이동
2. 한칸씩 이동하면서 현재 문자열보다 긴 팰린드롬인지 양쪽으로 확장하면서 비교 (투 포인터)

# References
> 파이썬 알고리즘 인터뷰 - 박상길

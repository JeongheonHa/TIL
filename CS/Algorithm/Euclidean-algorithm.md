# 유클리드 호제법

## 1. 최대공약수, 최소 공배수

유클리드 호제법을 이해하기 위해 중학교 때 배운 **최대 공약수(GCD, Greatest Common Division)** 와 **최소 공배수(LCM, Least Common Multiple)** 를 다시 되새겨보자.

- `최대 공약수(GLD)` : 두 수 A, B가 있을 때 두 수를 소인수 분해한 뒤, 두 수의 공통된 소인수를 모두 곱한 것 (즉, 약수 중 최대 값인 것을 말한다.)

- `최소 공배수(LCM)` : 두 수 A, B가 있을 때  두 수를 소인수 분해한 뒤, 두 수의 공통된 소인수와 나머지를 모두 곱한 것 (즉, 두 수의 공통된 배수 중 가장 작은 배수를 말한다.)

최대 공약수를 G라고 하고 최소 공배수를 L이라고 할 때 두 수 A, B와 G, L의 관계는 다음과 같다.</br></br>

### <center>AB = LG</center>

</br>두 수의 소인수들을 벤-다이어그램을 이용해 정렬해보면 쉽게 이 관계가 성립함을 확인해볼 수 있다.</br></br>

이제 유클리드 호제법을 이해하기 위한 최대 공약수와 최소 공배수를 복습해 보았으니 유클리드 호제법이 무엇인지 알아보자.</br></br>

## 2. 유클리드 호제법

두 수가 작은 경우 지금 까지의 방식을 이용해서 최대 공약수를 구할 수 있지만 두 수가 굉장히 큰 수일 경우에는 소인수 분해로 구하는 것보다 유클리드 호제법을 이용해 컴퓨터로 구하는 것이 더 효율적이다.

</br>유클리드 호제법의 정의는 다음과 같다. (단, A > B, A, B > 0)</br></br>


### <center>A를 B로 나눈 몫을 Q라 하고, 나머지를 R이라 할 때, gcd(A,B) = gcd(B,R)이다.</center>

</br>간단한 예제를 통해 확인해 보자.</br></br>


Q. 1254와 582의 최대 공약수를 구하라.   
- 1254 = 582 x 2 + 90
- 582 = 90 x 6 + 42
- 90 = 42 x 2 + 6
- 42 = 6 x 7   

이 경우 4단계에서 나누어 떨어졌으므로 그 전 단계인 3단계의 나머지인 6이 최소 공배수가 된다.</br></br>

### 증명

![IMG_1078](https://user-images.githubusercontent.com/108064146/190845559-9d314134-5552-46aa-ab0c-7de4a0d84835.jpeg)
![IMG_1079](https://user-images.githubusercontent.com/108064146/190845592-e2447fc9-a5e5-4dc8-904e-0ddb92c1c023.jpeg)</br></br>

### 파이썬으로 구현

최대 공약수 :

```python
def gcd(a, b):
    while b > 0:
        a, b = b, a % b
    return a
```

최소 공배수 :

```python
def lcm(a, b);
    return a * b / gcd(a, b)
```
# References
<https://dimenchoi.tistory.com/46>   
<https://www.youtube.com/watch?v=J5Yl2kHPAY4>

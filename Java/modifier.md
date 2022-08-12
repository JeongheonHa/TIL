# 제어자
## 1. 제어자
- 접근 제어자 : public, protected, default, private (4개 중 하나만 사용가능)
- 그 외 : static, final, abstract, native, transient, synchronized, volatile, strictfp
- 순서 : 접근 제어자 -> 그 외

## 2. static
- 대상
  * 멤버변수 - static 변수가 된다.
  * 메서드 - static method가 된다.

## 3. final
- 대상
  * 클래스 : 자손이 없는 클래스가 된다.
  * 메서드 : 오버라이딩 x
  * 멤버변수 + 지역변수 : 값 변경 x (상수), 인스턴스 변수의 경우 생성자에서 초기화

## 4 abstract
- 대상
  * 클래스 : 추상메서드가 있으면 추상 클래스
  * 메서드 : header만 있고 body는 없는 메서드

## 접근 제어자
제어자 | 같은 클래스 | 같은 패키지 | 자손 클래스 | 전체 |
---|---|---|---|---|
public| o | o | o | o |
protected| o | o | o |
(default)| o | o |
private| o |

# Reference
> 자바의 정석 - 남궁성

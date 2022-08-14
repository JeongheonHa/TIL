# 다형성
## 1. 다형성
- 조상티입의 참조변수로 자손타입의 객체를 다룰 수 있는 것 (반대로 자손타입 참조변수 -> 조상타입 객체 x)

## 2. 참조변수의 형변환
- 사용할 수 있는 멤버의 개수를 조절하는 것 (주소값, 객체 안바뀜)
- 생략 또한 가능 (참조할 수 있는 개수범위 내에 있으며 가능, 개수 초과면 x)
- *객체는 그대로있고 형변환만 한다는 거 주의!!!*

## 3. instanceof 연산자
- 참조변수의 형변환 가능여부를 확인할 때 사용
- 가능하면 True 반환 (조상 또한 true가 반환된다는 점 주의하자)
- *형변환 전에 **반드시** instanceof로 확인*

## 4. 참조변수와 인스턴스변수의 연결
- 멤버변수가 중복정의된 경우, 참조변수의 타입에 따라 연결되는 멤버변수가 달라진다. (참조변수에 영향 받음)
- 메서드의 중복의 경우, 참조변수의 타입에 관계없이 항상 실제 인스턴스의 타입에 정의된 메서드가 호출된다. (참조변수의 영향받지 않음)

## 5. 매개변수의 다형성 (장점 1)
- 참조형 매개변수는 메서드 호출시, 자신과 같은 타입 or 자손타입의 인스턴스를 넘겨줄 수 있다.

## 6. 여러 종류의 객체 하나의 배열로 다루기 (장점 2)
- 조상타입의 배열에 자손들의 객체를 담는 것 가능.

# Reference
> 자바의 정석 - 남궁성
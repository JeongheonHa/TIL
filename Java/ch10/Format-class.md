# Format-class

## 1. DecimalFormat클래스
- `숫자`를 다양한 `형식`으로 출력할 수 있게 해준다. (`format()` : 숫자 -> 문자열)
```java
double number = 1234567.89;
// 다양한 형식의 DecimalFormat인스턴스 생성
DecimalFormat df = new DecimalFormat("#.#E0");
// 숫자를 해당 형식에 넣고 문자열로 반환 
String result = df.format(number);  // format() : 숫자티입 -> 문자열
```

- 특정 `형식`으로 되어 있는 문자열에서 `숫자`를 뽑아낼 수 있다. (`parse()` : 문자열 -> 숫자)
```java
// 특정 형식의 DecimalFormat인스턴스 생성
DecimalFormat df = new DecimalFormat("#,###.##");
// 해당 문자열을 DecimalFormat인스턴스에 형식에 맞게 숫자로 저장
Number num = df.parse("1,234,567.89");
// 래퍼클래스를 double로 형변환
double d = num.doubleValue();
```
> Integer.parseInt("123,456.78");와 같이 parse ~ ()는 ","가 포함된 문자열을 숫자로 변환하지 못한다.   
> 이럴 땐 DecimalFormat클래스를 사용한다.    

| 기호 | 의미 | 패턴 |
|---|---|---|
| 0 | 10진수(값이 없을 때 0) | 0 / 0.0 |
| # | 10진수(값이 없으면 무시) | # / #.# |
| . | 소수점 | #.# |
| - | 음수 부호 | -#.# / #.#- |
| + | 양수 부호 | +#.# / #.#+ |
| E | 지수 부호 | #E0 / 0E0 / #.#E0 / 0.0E0 |
| ; | 구분자 | #.##;#,###,## |
| % | 퍼센트 | #.#% |
| \u2030 | 퍼밀(퍼센트 x 10) | #.#\u2030 |
| \u00A4 | 통화 | \u00A4 |
| ' | escape문자 | '#'#,### / "#,### |
> 자세한 설명은 [oracle 링크](https://docs.oracle.com/javase/7/docs/api/java/text/DecimalFormat.html) 참조

## 2. SimpleDateFormat클래스
- `날짜와 시간`을 다양한 `형식`으로 출력할 수 있게 해준다. (`format()` : 날짜와 시간 -> 문자열)
```java
// 현재 날짜와 시간 정보를 갖는 Date객체 생성
Date today = new Date();
// 특정 형식의 SimpleDateFormat인스턴스 생성
SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd");
// Date타입의 날짜와 시간을 문자열로 변환
String result = df.format(today);
```

- 특정 `형식`으로 되어 있는 문자열에서 `날짜와 시간`을 뽑아낼 수 있다. (`parse()` : 문자열 -> 날짜와 시간)
```java
// yyyy년 MM월 dd일 -> yyyy/MM/dd 바꾸기

// 다형성을 이용해 DateFormat타입의 참조변수(df)로 접근하는 특정 형식의 SimpleDateFormat인스턴스 생성
DateFormat df = new SimpleDateFormat("yyyy년 MM월 dd일");
// 다형성을 이용해 DateFormat타입의 참조변수(df2)로 접근하는 특정 형식의 SimpleDateFormat인스턴스 생성
DateFormat df2 = new SimpleDateFormat("yyyy/MM/dd");
// DateFormat타입의 참조변수로 접근한 SimpleDateFormat인스턴스에 parse를 이용해 날짜와 시간 문자열을 날짜와 시간으로 반환 후 Date타입으로 저장
Date d = df.parse("2022년 8월 17일");
// Date타입의 날짜와 시간을 df2가 접근한 형식에 맞게 문자열로 반환 후 String타입에 저장
String result = df2.format(d);
```

| 기호 | 의미 | 기호 | 의미
|---|---|---|---|
| G | 연대(BC,AD) | a | AM/PM |
| y | 년도 | h | 시간(0 ~ 23) |
| M | 월(1 ~ 12 or 1월 ~ 12월) | H | 시간(1 ~ 12) |
| w | 년의 몇 번째 주(1 ~ 53) | k | 시간(1 ~ 24) |
| W | 월의 몇 번째 주(1 ~ 5) | m | 분(0 ~ 59) |
| D | 년의 몇 번째 일(1 ~ 366) | s | 초(0 ~ 59) |
| d | 월의 몇 번째 일(1 ~ 31) | S | 천분의 일초(0 ~ 999) |
| F | 월의 몇 번째 요일(1 ~ 5) | z | Time zone(General time zone) |
| E | 요일 | Z | Time zone(RFC 822 time zone) |
| ' | escape문자(특수문자를 표현하는데 사용) | | |

## 3. ChoiceFormat클래스
- 특정 범위에 속하는 값을 문자열로 변환
```java
// 기준 점수를 배열로 저장
double[] limits = {60,70,80,90};
// 기준 학점을 배열로 저장
String[] grades = {"D","C","B","A"};
// 성적을 배열에 저장
int[] scores = {100,95,88,70,53,61,70};
// 각각의 원소들에 맞게 기준 점수와 기준 학점을 매칭시킨 ChoiceFormat인스턴스 생성
ChoiceFormat form = new ChoiceFormat(limits,grades);
// 성적의 갯수만큼 순환
for(int i = 0; i < scores.length; i++) {
    // format()을 이용해 성적(`숫자`)을 해당 범위의 기준 학점(`문자열`)으로 반환
    System.out.println(Scores[i] + ":" + form.format(sroce[i]));
}

// `주의` : 기준이 되는 경계값 double형 배열은 반드시 `오름차순`이어야한다.
```
### 3.1 기준 점수와 기준 학점을 동시에 정의하는 방법
- `limit#value` 형식을 따른다.
- `"#"`을 사용할경우 경계값 포함 o, `"<"`을 사용할 경우 경계값 포함 x
```java
// 기준 점수(숫자), 학점(문자열) 동시에 선언
String pattern = "60#D|70#C|80#B|90#A";
int[] scores = {91,90,80,88,70,54,65};

ChoiceFormat form = new ChoiceFormat(pattern);

for(int i = 0; scores.length; i++) {
    System.out.println(score[i] + ":" + form.format(scores[i]));
}
```

## 4. MessageFormat클래스
- 데이터를 정해진 양식에 맞춰 출력할 수 있도록 도와준다.
- `{숫자}`로 표현된 부분이 데이터가 출력될 자리이다.
```java
// 기본 메세지 양식 생성
String msg = "Name: {0} \nTel: {1} \nAge: {2} \nBirthday: {3}";
// 추가할 데이터의 객체 배열 생성
Object[] arguments = {
    "하정헌", "010-1234-5678", "22", "11-11"
};
// {숫자}부분에 추가한 데이터를 넣고 문자열로 저장
String result = MessageFormat.format(msg, arguments);
System.out.println(result);
```
# References
> 자바의 정석 - 남궁성   
> Link :<https://docs.oracle.com/javase/7/docs/api/java/text/DecimalFormat.html>

# Calendar & Date

## 1. 배경
- `java.util.Date`   
: `jdk1.0`에 날짜와 시간을 다룰 목적으로 만들어진 클래스
  > 대부분 앞으로 사용하지말 것을 권장하지만 여진히 쓰인다.

- `java.util.Calendar`   
: `jdk1.1`에 추가된 Date클래스를 개선한 클래스

- `java.time`   
: `jdk1.8`에 추간된 Date와 Calendar의 단점을 개선한 클래스

  > Date와 Calendar는 날짜와 시간을 함께 다뤄야한다는 단점이 존재한다.     
  > time은 날짜와 시간을 따로 다룰 수 있게 class를 나눠서 Date와 Calendar의 단점을 보완했다.

## 2. Calendar
- `java.util.Calendar`   
: 추상 클래스이므로 getInstance()를 통해 구현된 객체를 얻어야 한다. 
- getInstance()는 시스템의 국가와 지역설정을 확인하여 Calendar클래스를 상속받아 구현된 알맞는 인스턴스를 반환한다. (우리는 GregorianCalendar)
```java
Calendar cal = Calendar.getInstance();
int thisYear = cal.get(Calendar.YEAR);
// 해당 월의 마지막 날 구하기
int lastDayOfMonth = cal.getActualMaximum(Calendar.DATE);
```

## 3. Calendar에 정의된 필드
| 날짜 필드명 | 설명 | 시간 필드명 | 설명 |
|---|---|---|---|
| YEAR | 년 | HOUR | 시간(0~11) |
| MONTH | 월(0부터) | HOUR_OF_DAY | 시간(0~23) | 
| DATE | 일 | MINUTE | 분 |
| WEEK_OF_YEAR | 1월1일부터 몇 번째 주 | SECOND | 초 |
| WEEK_OF_MONTH | 그 달의 몇 번째 주 | MILLISECOND | 천분의 일초 |
| DAY_OF_MONTH | 그 달의 일 | ZONE_OFFSET | GMT기준 시차(천분의 일초 단위) |
| DAY_OF_YEAR | 1월 1일부터 몇 일 | AM_PM | 오전/오후 |
| DAY_OF_WEEK | 요일 | | |
| DAY_OF_WEEK_IN_MONTH | 그 달의 몇 번째 요일 | | |

## 4. set() 
- 날짜와 시간을 지정하는 메서드
```java
// 특정 필드를 해당 값으로 변경
void set(int field, int value)
// 년,월,일,시,분,초 원하는 값으로 변경
void set(int year, int month, int date)
void set(int year, int month, int date, int hourOfDay, int minute)
void set(int year, int month, int date, int hourOfDay, int minute, int second)
```
> 월은 0부터 시작한다는 점을 주의하자 !!  

- 날짜 지정 예
```java
Calendar date1 = Calendar.getInstance();
date1.set(2022, 7, 17); // 월은 0부터 시작한다는 점을 주의하자 !!
```

- 시간 지정 예
```java
Calendar time1 = Calendar.getInstance();
time1.set(Calendar.HOUR_OF_DAY, 10);
time1.set(Calendar.MINUTE, 20);
time1.set(Calendar.SECOND, 30);
```
> 날짜는 매개변수를 한번에 받아올 수 있지만 set()메서드에는 시분초만을 한번에 설정하지 못하기 때문에 각각 설정해 주어야 한다.   
> 두 날짜간의 차이를 알고 싶으면 getTimeInMillis()를 이용해 밀리세컨드로 바꾼 후 1/1000을 곱해 초 단위로 바꿔 계산한다.

## 5. clear()
- clear()는 Calendar객체의 모든 필드를 1970/Jan/01/00/00/00/로 초기화 한다.
- 무조건 이 순서를 따르는 것이 좋다. 
```java
Calendar dt = Calendar.getInstance();
dt.clear();
dt.set( 년도 , 날짜 , ... );
```
> clear()를 안할 경우 두 시간의 차를 구할 때 getTimeInMillis()를 하는 과정에서 오차가 발생하여 정확한 값을 얻을 수 없다.

## 6. clear(int field)
- Calendar개체의 특정 필드를 1970/Jan/01/00/00/00/에 맞게 초기화
```java
Calendar dt = Calendar.getInstance();
dt.clear(Calendar.SECOND);
dt.clear(Calendar.MINUTE);
...
```

## 7. add()
- 특정 필드의 값을 증가 or 감소 (`주의` : 다른 필드에 영향을 준다)
```java
// Calendar를 상속받아 구현된 GregorianCalendar인스턴스 생성
Calendar date = Calendar.getInstance();
// 초기화
date.clear();
// 설정
date.set(2020,7,31);        // 2020/08/31

// 추가
date.add(Calendar.DATE,1);  // 2020/09/01
date.add(Calendar.MONTH, -8);
```
## 8. roll()
- 특정필드의 값을 증가 or 감소 (`주의` : 다른 필드에 영향을 주지 않는다)
```java
date.set(2020,7,31);        // 2020/08/31

date.roll(Calendar.DATE,1); // 2020/08/01
```

## 9. Date와 Calendar간의 변환
- Calendar -> Date
```java
Calendar cal = Calendar.getInstance();
...
Date d = new Date(cal.getTimeInMillis());
```

- Date -> Calendar

```java
Date d = new Date();
...
Calendar cal = Calendar.getInstance();
cal.setTime(d);
```

# Reference
> 자바의 정석 - 남궁성

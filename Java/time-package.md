# time패키지

## 1. java.time패키지
- 이 패키지에 속한 클래스들은 모두 `불변`이다.
- `jdk1.8`부터 추가되었다.
- time패키지는 4개의 하위패키지를 가진다.

| 패키지 | 설명 |
|---|---|
| java.time | 날짜와 시간을 다루는데 필요한 핵심 클래스 제공 |
| java.time.chrono | 표준(ISO)이 아닌 달력 시스템을 위한 클래스 제공 |
| java.time.format | 날짜와 시간을 파싱하고, 형식화하기 위한 클래스 제공 |
| java.time.temporal | 날짜와 시간의 필드와 단위를 위한 클래스 제공 |
| java.time.zone | 시간대(time-zone)와 관련되 클래스 제공 |

> 불변 : 날짜나 시간을 변경하는 메서드들은 기존의 객체를 변경하는 대신 항상 변경된 새로운 객체를 반환한다.   
> 불변인 이유 : 멀티 쓰레드환경에서 동시에 여러 쓰레드가 같은 객체에 접근하면 데이터가 잘못될 수 있기 때문에 (쓰레드가 안전하지 않다.)

## 2. java.time의 핵심 클래스
- `LocalDate` : 날짜 
- `LocalTime` : 시간 
- `LocalDateTime` : 날짜와 시간이 모두 필요할 때 사용
- `ZonedDateTime` : 여기에 시간대까지 추가하고 싶을 때 사용
- `Period` : 날짜 간의 차이를 표현할 때 사용
- `Duration` : 시간의 차이를 표현할 때 사용
> java.time패키지에서는 `날짜`와 `시간`을 별도의 클래스로 분리

## 3. 핵심 클래스가 구현한 `인터페이스`
- `Temporal`,`TemporalAccessor`, `TemporalAdjuster` : 날짜와 시간을 표현하는 클래스들이 구현하는 인터페이스들   
-> LocalDate, LocalTime, LocalDateTime, ZonedDateTime, Instant 등
- `TemporalAmount` : 날짜와 시간의 차이를 표현하는 클래스가 구현하는 인터페이스   
-> Period, Duration
- `TemporalUnit` : 날짜와 시간의 단위를 정의해 놓은 인터페이스 (구현한 클래스 : 열겨형`ChronoUnit`)
- `TemporalField` : 년, 월, 일 등 날짜와 시간의 필드를 정의해 놓음 (구현한 클래스 : 열거형`ChronoField`)

## 4. get() & plus() & minus()
- `get()` : 날짜와 시간에서 특정 필드의 값만을 얻을 때 사용 (getXXX()도 가능) 
- `plus()`/`minus()` : 특정 날짜와 시간에서 지정된 단위의 값을 더하거나 뺄 때 사용 (plusXXX()도 가능)
```java
// 정의
int get(TemporalField field)
LocalDate plus(long amountToAdd, TemporalUnit unit)
```
```java
// get()
LocalTime now = LocalTime.now();
int minute = now.getMinute();
int minute = now.get(ChronoField.MINUTE_OF_HOUR);
// plus()
LocalDate today = Local.Date.now();
LocalDate tomorrow = today.plus(1, ChronoUnit.DAYS);
LocalDate tomorrow = today.plusDays(1);
```

## 5. LocalDate & LocalTime
- `now()`는 현재 날짜 시간을 지정, `of()`는 특정 날짜 시간을 지정
- 일 단위나 초 단위로도 지정가능 (1day = 24 * 60 * 60 = 86400s)
- 둘다 static메서드이다.
```java
// now() : 현재날짜와 시간을 저장하는 객체를 생성하는 메서드
LocalDate date = LocalDate.now();
LocalTime time = LocalTime.now();
LocalDateTime dateTime = LocalDateTime.now();
ZonedDateTime dateTimeInKr = ZonedDateTime.now();
// of() : 해당 필드의 값을 순서대로 지정해 객체를 생성하는 메서드
LocalDate date = LocalDate.of(2022, 8, 18);
LocalDate time = LocalTime.of(23, 59, 59)
LocalDate birthTime = LocalTime.of(86399);
LocalDateTime dateTime = LocalDateTime.of(date, time);
ZoneDateTime zDateTime = ZoneDateTime.of(dateTime, ZoneId.of("Asia/Seoul"));
```

- `of()`의 여러 버전
```java
static LocalDate of(int year, Month month, int dayOfMonth)
static LocalDate of(int year, int month, int daOfMonth)

static LocalTime of(int hour, int min)
static LocalTime of(int hour, int min, int sec)
static LocalTime of(int hour, int min, int sec, int nanoOfSecond)
```

- `parse()`로 문자열을 LocalDate나 LocalTime으로 변환할 수 있다.
```java
LocalDate curDate = LocalDate.parse("2022-8-17");
LocalTime curTime = LocalTime.parse("23:59:59");
```

## 6. LocalDate & LocalTime 필드 값 가져오기
- `get()`, `getXXX()` : 특정 필드의 값을 가져올 수 있는 메서드 (int 위주)
- `getLong()` : 특정 필드의 값을 가져올 수 있는 메서드 (int의 범위를 넘어갈 경우)
- `주의` : Calendar와 달리 month의 범위는 1 ~ 12 이고 요일은 월요일이 1, 일요일이 7이다.

### 6.1 LocalDate의 필드 값을 반환하는 메서드
| 메서드 | 설명(2022-12-31 23:59:59) |
|---|---|
| int getYear() | 년도(2022) |
| int getMonthValue() | 월(12) |
| Month getMonth() | 월(DECEMBER), getMonth().getValue() = 12 |
| int getDayOfMonth() | 일(31) |
| int getDayOfYear() | 같은 해의 1월 1일부터 몇번째 일(365) |
| DayOfWeek getDayOfWeek() | 요일(FRIDAY), getDayOfWeek().getValue() = 5 |
| int lengthOfMonth() | 같은 달의 총 일수(31) |
| int lengthOfYear() | 같은 해의 총 일수(365), 윤년의 경우(366) |
| boolean isLeapYear() | 윤년여부 확인(false) |

### 6.2 LocalTime의 필드 값을 반환하는 메서드
| 메서드 | 설명(2022-12-31 23:59:59) |
|---|---|
| int getHour() | 시(23) |
| int getMinute() | 분(59) |
| int getSecond() | 초(59) |
| int getNano() | 나노초(0) |

### 6.3 get() & getLong()
```java
// 정의

// get()
int get(TemporalField field)
// getLong()
long getLong(TemporalField field)
```
### 6.4 get(), getLong()에 매개변수로 사용할 수 있는 필드 목록 (*는 getLong()에 사용)
| TemporalField(ChronoField) | 설명 | TemporalField(ChronoField) | 설명 |
|---|---|---|---|
| ERA | 시대 | EPOCH_DAY* | EPOCH(1970.1.1)부터 몇번째 날 |
| YEAR_OF_ERA or YEAR | 년 | MINUTE_OF_DAY | 그 날의 몇 번째 분 |
| MONTH_OF_YEAR | 월 | SECOND_OF_DAY | 그 날의 몇 번째 초 |
| DAY_OF_WEEK | 요일(1 : 월요일) | MILLI_OF_DAY | 그 날의 몇 번째 밀리초 |
| DAY_OF_MONTH | 일 | MICRO_OF_DAY* | 그 날의 몇 번째 마이크로초 |
| AMPM_OF_DAY | 오전/오후 | NANO_OF_DAY* | 그 날의 몇 번째 나노초 |
| HOUR_OF_DAY | 시간(0~23) | ALIGNED_WEEK_OF_MONTH | 그 달의 n번째 주 |
| CLOCK_HOUR_OF_DAY | 시간(1~24) | ALIGNED_WEEK_OF_YEAR | 그 해의 n번째 주 |
| HOUR_OF_AMPM | 시간(0~11) | ALIGNED_DAY_OF_WEEK_IN_MONTH | 요일(그 달의 1일을 월요일로 간주하여 계산) |
| CLOCK_HOUR_OF_AMPM | 시간(1~12) | ALIGNED_DAY_WEEK_IN_YEAR | 요일(그 해의 1월 1일을 월요일로 간주하여 계산) |
| MINUTE_OF_HOUR | 분 | INSTANT_SECONDS | 년월일을 초단위로 환산(Instant에서만 사용가능) |
| SECOND_OF_MINUTE | 초 | OFFSET_SECONDS | UTC와의 시차(ZoneOffset에만 사용가능) |
| MILLI_OF_SECOND | 천분의 일초 | PROLEPTIC_MONTH | 년월을 월단위로 환산(2022년8월 -> 2022*12+8)
| MICRO_OF_SECOND* | 백만분의 일초 | | |
| NANO_OF_SECOND* | 10억분의 일초 | | |
| DAY_OF_YEAR | 그 해의 몇번째 날 | | |
> 사용할 수 있는 필드는 클래스마다 다르다는 점을 주의하자 !!

## 7. 특정 필드값 변경하기
- `with()`, `plus()`, `minus()`로 특정 필드의 값 변경(새로운 객체가 반환)
- `with()` : 원하는 필드를 직접 지정
- `plus()`/`minus()` : 특정 필드에 값을 더하거나 뺀다.
```java
// 정의
LocalDate with(TemporalField field, long newValue)

LocalTime plus(TemporalAmount amountToAdd)
LocalTime plus(long amountToAdd, TemporalUnit unit)
LocalDate plus(TemporalAmount amountToAdd)
LocalDate plus(long amountToAdd, TemporalUnit unit)
```
> `주의` : 필드를 변경하는 메서드들은 항상 새로운 객체를 생성해서 반환하기 때문에 대입 연산자를 같이 사용해야한다.

- LocalTime의 `truncatedTo()`는 지정된 것보다 작은 단위의 필드를 0으로 만든다. 
- LocalDate의 필드인 년, 월, 일은 0이될 수 없기 때문에 LocalDate에는 truncatedTo()메서드가 없다.

### 7.1 열거형 ChronoUnit에 정의된 상수 목록
| TemporalUnit(ChronoUnit) | 설명 |
|---|---|
| FOREVER | Long.AX_VALUE초(약 3천억년) |
| ERAS | 1,000,000,000년 |
| MILLENNIA | 1000년 |
| CENTURIES | 100년 |
| DECADES | 10년 |
| YEARS | 년 |
| MONTHS | 월 |
| WEEKS | 주 |
| DAYS | 일 |
| HALF_DAYS | 반나절 |
| HOURS | 시 |
| MINUTES | 분 |
| SECONDS | 초 |
| MILLIS | 천분의 일초 |
| MICROS | 백만분의 일초 |
| NANOS | 10억분의 일초 |

## 8. 날짜와 시간 비교
- `isAfter()`, `isBefore()`, `isEqual()`를 사용하여 날자와 시간을 비교할 수 있다.
```java
boolean isAfter (ChronoLocalDate other)
boolean isBefore (ChronoLocalDate other)
boolean isEqual (ChronoLocalDate other) // LocalDate에만 있다.
```

- LocalDate와 LocalTime도 `compareTo()`가 적절히 오버라이딩되어 있어 compareTo()를 이용해 비교할 수 있지만   
위의 3개의 메서드를 사용하면 보다 편리하게 비교할 수 있다.
```java
// compareTo를 이용하여 비교
int result = date1.compareTo(date2);    // 같으면 0, date1이 이전이면 -1, date1이 이후면 1
```

- `equals()` vs `isEqual()`
  + 연표가 다른 두 날짜를 비교하기 위해서 isEqual()을 제공한다.
  + 모든 필드가 일치해야하는 equals()에 비해 isEqual()은 날짜만 비교한다.
  + 대부분의 경우 equals()와 isEqual()는 결과가 같기 때문에 isEqual() 대신 equals()를 사용해도 된다.

## 9. Instant
- 에포크 타입(EPOCH TIME, 1970-01-01 00:00:00 UTC)부터 경과된 시간을 `나노초 단위`로 표현
- java.util.Date를 대체한다.
- 날짜와 시간과 달리 단일 진법(10진법)이라 계산에 편리하다. (사람이 사용하는 날짜와 시간은 여러 진법이 섞여있어서 계산하기 어렵다.)
- `now()`와 `ofEpochSecond()`를 사용한다.
```java
Instant now = Instant.now();
Instant now2 = Instant.ofEpochSecond(now.getEpochSecond());
Instant now3 = Instant.ofEpochSecond(now.getEpochSecond(), now.getNano());
```

- 나노초 단위가 아닌 `밀리초 단위`의 에포크 타임이 필요할 경우
```java
long toEpochMilli()
```

- Instant와 Date간의 변환에 사용하는 메서드 (jdk.1.8추가)
```java
// Instant -> Date
static Date from(Instant instant)
// Date -> Instant
Instant toInstant()
```

## 10. LocalDateTime & ZonedDateTime
- `LocalDate` + `LocalTime` -> `LocalDateTime`
- `LocalDateTime` + `시간대` -> `ZonedDateTime`

### 10.1 LocalDate + LocalTime -> LocalDateTime
```java
LocalDate date = LocalDate.of(2022, 12, 31);
LocalTime time = LocalTime.of(12,34,56);
// 전체 합치기
LocalDateTime dt = LocalDateTime.of(date, time);                // (1)
LocalDateTime dt2 = date.atTime(time);                          // (2)
LocalDateTime dt3 = Time.atDate(date);                          // (3)
LocalDateTime dt4 = date.atTime(12, 34, 56);                    // (4)
LocalDateTIme dt5 = time.atDate(LocalDate.of(2022, 12, 31));    // (5)
LocalDateTime dt6 = date.atStartOfDay();                        // (6)

// LocalDateTime클래스에도 now()와 of()메서드가 정의되어있다.
```

### 10.2 LocalDateTime -> LocalDate + LocalTime
```java
LocalDateTime dt = LocalDateTime.of(2022, 12, 31, 12, 34, 56);
// LocalDateTime -> LocalDate
LocalDate date = dt.toLocalDate();
// LocalDateTime -> LocalTime
LocalTime time = dt.toLocalTime();
```

### 10.3 LocalDateTime -> ZonedDateTime
- 기존에는 TimeZone클래스로 시간대를 다뤘지만 time패키지에서는 `ZoneId`라는 클래스를 사용한다.
- LocalDate에 시간 정보를 추가하는 atTime()을 쓰면 LocalDateTime을 얻을 수 있는 것 처럼   
LocalDateTime에 `atZone()`으로 시간대 정보를 추가하면, ZonedDateTime을 얻을 수 있다.
 > 사용가능한 ZoneId의 목록은 ZoneId.getAvailableZoneIds()로 얻을 수 있다.
```java
// 일반적인 방법
ZoneId zid = ZoneId.of("Asia/Seoul");
ZonedDateTime zdt = dateTime.atZone(zid);
System.out.println(zdt);

// LocalDate에 있는 atStartOfDay()사용
ZonedDateTime zdt = LocalDate.now().atStartOfDay(zid);
System.out.println(zdt);

// 현재 특정 시간대의 시간을 알고 싶을 때
ZoneId nyId = ZoneId.of("America/New_York");
ZonedDateTime nyTime = ZonedDateTime.now().withZoneSameInstant(myId);   // of()를 사용할 시 날짜와 시간 지정
```

### 10.4 ZoneOffset
- UTC로부터 얼마만큼 떨어져 있는지를 `ZoneOffset`으로 표현
```java
// 현재 UTC로부터 얼마나 떨어져있는지 불러온다.
ZoneOffset krOffset = ZonedDateTime.now().getOffset();
// UTC로부터 떨어져있는 시간을 지정한다.
ZoneOffset krOffset = ZoneOffset.of("+9");
// 초 단위로 바꿔서 저장한다.
int krOffsetInSec = krOffset.get(ChronoField.OFFSET_SECONDS);
```

### 10.5 OffsetDateTime
- `ZonedDateTime`클래스가 구역을 표현하기위해 `ZoneId`클래스를 사용하는 것 처럼   
ZoneId가 아닌 `ZoneOffset`을 사용하는 것이 `OffsetDateTime`이다.
- `ZoneId` vs `ZoneOffset`
  + ZoneId : 일광 절약시간처럼 시간대와 관련되 규칙들을 포함한 클래스 (불안전)
  + ZoneOffset : 시간대를 시간의 차이로만 구분 (안전)
  + 같은 지역내의 컴퓨터간에 데이터를 주고받을 때는 ZoneId로도 충분하지만 서로 다른 시간대에 컴퓨터간의     
    데이터를 주고받을 때는 일관된 시간체계를 유지하는 OffsetDateTime을 사용하도록 하자 !!
```java
// ZonedDateTime에서 
ZonedDateTime zdt = ZonedDateTime.of(date, time, zid);           // ZoneId zid = ZoneId.of("Asia/Seoul");
// <1> OffsetDateTime에서
OffsetDateTime odt = OffsetDateTime.of(date, time, krOffset);   // ZoneOffset krOffset = ZoneDateTime.now().getOffset();
// <2> ZonedDateTime -> OffsetDateTime
OffsetDateTime odt = zdt.toOffsetDateTime();
```
> 일광 절약시간(DST, Dayligth Saving Time) : 계절별로 시간을 더했다 뺐다하는 방식

### 10.6 ZonedDateTime의 변환
- ZonedDateTime을 변환하는데 사용되는 메서드
  + `LocalDate toLocalDate()`
  + `LocalTime toLocalTime()`
  + `LocalDateTime toLocalDateTime()`
  + `OffsetDateTime toOffsetDateTime()`
  + `long toEpochSecond()`
  + `Instant toInstant()`

- `ZonedDateTime`을 `GregorianCalendar`로 변환하는 방법
  + GregorianCalendar와 가장 유사한 것이 ZonedDateTime이며 두 클래스간의 변환방법을 알면    
  위에 나열된 메서드를 이용해서 다른 날짜와 시간 클래스들로 변환할 수 있다.
```java
// ZonedDateTime -> GregorianCalendar
GregorianCalendar from(ZonedDateTime zdt)
// GregorianCalendar -> ZonedDateTime
ZonedDateTime toZonedDateTime()
```

## 11. TemporalAdjusters
- plus(), minus()로 계산하기 복잡한 날짜계산을 도와주는 클래스
```java
// 예시
LocalDate today = LocalDate.now();
LocalDate nextMonday = today.with(TemporalAdjusters.next(DayOfWeek.MONDAY));
```

### 11.1 TemporalAdjusters의 메서드
| 메서드 | 설명 |
|---|---|
| firstDayOfNextYear() | 다음 해의 첫 날 |
| firstDayOfNextMonth() | 다음 달의 첫 날 |
| firstDayOfYear() | 올 해의 첫 날 |
| firstDayOfMonth() | 이번 달의 첫 날 |
| lastDayOfYear() | 올 해의 마지막 날 |
| lastDayOfMonth() | 이번 달의 마지막날 |
| firstInMonth(DayOfWeek dayOfWeek) | 이번 달의 첫 번째 ?요일의 날짜 |
| lastInMonth(DayOfWeek dayOfWeek) | 이번 달의 마지막 ?요일의 날짜 |
| previous(DayOfWeek dayOfWeek) | 지난 ?요일의 날짜(당일 미포함) |
| previousOrSame(DayOfWeek dayOfWeek) | 지난 ?요일의 날짜(당일 포함) |
| next(DayOfWeek dayOfWeek) | 다음 ?요일의 날짜(당일 미포함) |
| nextOrSame(DayOfWeek dayOfWeek) | 다음 ?요일의 날짜(당일 포함) |
| dayOfWeekInMonth(int ordinal, DayOfWeek dayOfWeek) | 이번 달의 n번째 ?요일의 날짜 |

## 12. Period & Duration
- `Period` : 날짜의 차이 계산하기위한 클래스
- `Duration` : 시간의 차이 계산하기위한 클래스
- 두 날짜 or 시간의 차이를 구할 때는 `between()`을 사용
```java
// 두 날짜간의 차이 계산
LocalDate date1 = LocalDate.of(2021,1,1);
LocalDate date2 = LocalDate.of(2022,12,31);

Preiod pe = Period.between(date, date2);

// 두 시간간의 차이 계산
LocalTime time1 = LocalTime.of(00,00,00);
LocalTime time2 = LocalTime.of(12,34,56);

Duration du = Duration.between(time1, time2);
```

- 특정 필드의 값을 얻을 때는 `get()`을 사용
```java
long year = pe.get(ChronoUnit.YEARS);   // int getYears()도 가능
long month = pe.get(ChronoUnit.MONTHS); // int getMonths()도 가능
long day = pe.get(ChronoUnit.DAYS);     // int getDays()도 가능

long sec = du.get(ChronoUnit.SECONDS);  // long getSeconds도 가능
int nano = du.get(ChronoUnit.NANOS);    // int getNano()도 가능
```

- 시간을 구하는 `getHours()`, `getMinute()`같은 메서드는 없다.
```java
// Preiod인스턴스의 유닛을 얻는다.
System.out.println(pe.getUnits());  // [Years, Months, Days] <- 이것만 사용 가능
// Duration인스턴스의 유닛을 얻는다.
System.out.println(du.getUnits());  // [Seconds, Nanos] <- 이것만 사용 가능
```

- 시분초를 구할 때는 `Duration`을 `LocalTime`으로 변환한다.
```java
// LocalTime인스턴스에 00:00으로 초기화한 후 Duration인스턴스의 초 차이를 가져와 더해준다. 
LocalTime gapTime = LocalTime.of(0,0).plusSeconds(du.getSeconds());
// LocalTime인스턴스에 저장된 시간, 분, 초를 얻는다.
int hour = gapTime.getHour();
int min = gapTime.getMinute();
int sec = gapTime.getSecond();
// Duration인스턴스의 나노 초를 얻는다.
int nano = du.getNano();
```

### 12.1 between() & until()
- between()과 until()은 거의 같은 기능을 한다.
- 단, `between()`은 `static`메서드이고, `until()`은 `인스턴스`메서드이다.
- Period클래스는 년,월,일로 분리해서 저장하므로, D-day를 구할 때는 until()을 쓰도록 하자.
```java
Period pe = today.until(myBirthday);
long dday = today.until(myBirthday, ChronoUnit.DAYS);
```
- LocalTime에도 until()이 있지만, Duration을 반환하는 until()은 없다.
```java
long sec = LocalTime.now().until.(endTime. ChronoUnit.SECONDS);
```

### 12.2 of(), ofXXX(), with(), withXXX()
- ofXXX() : 특정 필드 지정
  + Period : `of()`, `ofYears()`, `ofMonths()`, `ofWeeks()`, `ofDays()`
  + Duration : `of()`, `ofDays()`, `ofHours()`, `ofMinutes()`, `ofSeconds()`

- withXXX() : 특정 필드 변경
  + Period : `with()`, `withYears()`, `withMonths()`, `withDays()`
  + Duration : `with()`, `withSeconds()`, `withNanos()`

### 12.3 사칙연산, 비교연산, 기타 메서드
- plus(), minus()외의 곱셈과 나눗셈을하는 메서드도 존재(`multipliedBy()`, `dividedBy()`)
```java
// 1년을 빼고 2를 곱한다.
pe = pe.minusYears(1).multipliedBy(2);
// 1년을 더하고 60으로 나누다.
du = du.plusHours(1).dividedBy(60);

// Period에는 나눗셈을 위한 메서드는 존재하지 않는다 (유용하지 않기 때문에)
```
- `isNegative()`와 `isZero()`를 사용하면 날짜와 시간의 순서를 확인 가능
```java
// 날짜의 차가 0인지 확인
boolean sameDate = Period.between(date1, date2).isZero();
// 시간의 차가 음수인지 확인
boolean isBefore = Duration.between(time1, time2).isNegative();
```
- Period에는 abs()가 없기 때문에 `negated()`를 이용해 절대값을 표현하다.
```java
// 시간의 차가 음수인지 확인
if(du.isNegative())
    // 음수일 경우, 부호를 반대로 바꿔서 저장
    du = du.negated();
```

### 12.4 다른 단위로 변환
- `get()`은 특정 필드의 값을 그대로 가져오지만, `toXXX()`는 해당 단위로 변환해서 반환한다.

| 클래스 | 메서드 | 설명 |
|---|---|---|
| Period | long toTotalMonths() | 년월일을 월 단위로 변환해서 반환 (일단위는 무시) |
| Duration | long toDays() | 일 단위로 변환해서 반환 |
|| long toHours() | 시간 단위로 변환해서 반환 |
|| long toMinutes() | 분 단위로 변환해서 반환 |
|| long toMills() | 천분의 일초 단위로 변환해서 반환 |
|| long toNanos() | 나노 초 단위로 변환해서 반환 |

## 13. 파싱(parsing)과 포맷(format)
- parsing : 날짜와 시간을 원하는 형식으로 출력하고 해석하는 방법
- 형식화와 관련된 클래스는 java.time.format패키지에 들어있으며    
그 중 `java.time.format.DateTimeFormatter` 클래스에 자주 쓰이는 기본적인 형식이 저장되어 있다.
```java
// LocalDate인스턴스에 해당 년,월,일 지정
LocalDate date = LocalDate.of(2022,1,1);
// LocalDate인스턴스의 년,월,일을 DateTimeFormatter클래스의 format()를 이용해 ISO_LOCAL_DATE형식으로 바꾼다
String yyyyMMdd = DateTimeFormatter.ISO_LOCAL_DATE.format(date);  // "2022-01-01"
// LocalDate클래스의 format()을 이용하여 ISO_LOCAL_FATE 형식으로 바꾼다.
String yyyyMMdd = date.format(DateTimeFormatter.ISO_LOCAL_DATE);  // "2022-01-01"

// format()메서드는 LocalDate나 LocalTime같은 클래스에도 존재한다.
```
- DateTimeFormatter에 상수로 정의된 형식

| DateTimeFormatter | 보기 |
|---|---|
| ISO_DATE_TIME | 2011-12-03T10:15:30+01:00[Europe/Paris] |
| ISO_LOCAL_DATE | 2011-12-03 |
| ISO_LOCAL_TIME | 10:15:30 |
| ISO_LOCAL_DATE_TIME | 2011-12-03T10:15:30 |
| ISO_OFFSET_DATE | 2011-12-03+01:00 |
| ISO_OFFSET_TIME | 10:15:30+01:00 |
| ISO_OFFSET_DATE_TIME | 2011-12-03T10:15:30+01:00 |
| ISO_ZONED_DATE_TIME | 2011-12-03T10:15:30+01:00[Europe/Paris]
| ISO_INSTANT | 2011-12-03T10:15:30Z |
| BASIC_ISO_DATE | 20111203 |
| ISO_DATE | 2011-12-03+01:00 / 2011-12-03 |
| ISO_TIME | 10:15:30+01:00 / 10:15:30 |
| ISO_ORDINAL_DATE | 2012-337 |
| ISO_WEEK_DATE | 2012-W48_6 |
| RFC_1123_DATE_TIME | Tue, 3 Jun 2008 11:05:30 GMT |

### 13.1 로케일(locale)에 종속된 형식화
- DateTimeFormatter의 static메서드 `ofLocalizedDate()`, `ofLocalizedTime()`, `ofLocalizedDateTime()`은 로케일에 종속된 포맷터이다.
```java
DateTimeFormatter formatter = DateTimeFormatter.ofLocalizedDate(FormatStyle.SHORT);
String shortFormat = formater.format(LocalDate.now());
```
- 열거형 FormatStyle에 정의된 상수와 출력 예

| FormatStyle | 날짜 | 시간 |
|---|---|---|
| FULL | 2015년 11월 28일 토요일 | N/A |
| LONG | 2015년 11월 28일 (토) | 오후 9시 15분 13초 |
| MEDIUM | 2015.11.28 | 오후 9:15:13 |
| SHORT | 15.11.28 | 오후 9:15 |

### 13.2 출력형식 직접 정의하기
- DateTimeFormatter의 `ofPattern()`으로 직접 정의 가능
```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
```

### 13.3 문자열을 날짜와 시간으로 파싱하기
- `parse()`를 이용한다. (문자열 -> 날짜,시간)
```java
static LocalDateTime parse(CharSequence text)
static LocalDateTime parse(CharSequence text, DateTimeFormatter formatter)
```

- DateTimeFormatter에 정의된 형식을 사용할 경우
```java
// 기본적인 방법
LocalDate date = LocalDate.parse("2016-01-02", DateTimeFormatter.ISO_LOCAL_DATE);
// 자주 사용되는 형식은 생략 가능 (문자열만 인자로 받는다)
LocalDate date = LocalDate.parse("2016-01-02"); // (1)
LocalTime time = LocalTime.parse("23:59:59");   // (2)
LocalDateTime DateTime = LocalDateTime.parse("2001-01-01T23:59:59");  // (3)
```

- ofPattern()으로 파싱
```java
// ofPattern()을 이용해 직접 정의한 형식을 pattern에 저장
DateTimeFormatter pattern = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
// 문자열에 형식 적용
LocalDateTime endOfYear = LocalDateTime.parse("2015-12-31 23:59:59", pattern);
```

# Reference
> 자바의 정석 - 남궁성

# 스트림

## 1. 스트림
스트림은 데이터 소스를 추상화함으로써 데이터 소스가 무엇이든 같은 방식으로 다룰 수 있게 해준다.
- 배열, 컬렉션, 파일에 저장된 데이터 모두 같은 방식으로 다룰 수 있다.

### 1.1 스트림의 특징
- 스트림은 데이터 소스를 변경하지 않는다.
- 스트림은 일회용이다.
- 스트림은 작업을 내부 반복으로 처리한다.

### 1.3 병렬 스트림
- `parallel()`: 연산을 병렬로 처리(내부에서 fork&join을 사용)
- `sequential()`: 연산을 병렬로 처리하지 않는다. (기본값)
- 병렬 처리를 한다고 무조건 빠른 것은 아니다.

## 2. 스트림 생성

### 2.1 컬렉션
컬렉션의 최고 조상인 Collection에 stream()이 정의되어 있다.
```java
Stream<T> Collection.stream()

//예시
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
Stream<Integer> intStream = list.stream()
```

### 2.2 객체 배열
Stream과 Arrays에 static 메서드로 정의되어 있다.
```java
Stream<T> Stream.of(T... values) // 가변인자
Stream<T> Stream.of(T[])
Stream<T> Arrays.stream(T[])
Stream<T> Arrays.stream(T[] array, int start, int end)

Stream<String> strStream = Stream.of("a", "b", "c")
Stream<String> strStream = Stream.of(new String[] {"a", "b", "c"}) 
Stream<String> strStream = Arrays.stream(new String[] {"a", "b", "c"}) 
Stream<String> strStream = Arrays.stream(new String[] {"a", "b", "c"}, 0, 3)
```

### 2.3 기본형 배열
기본형 배열을 소스로 하는 스트림의 메서드들도 위와 같기 때문에 구분없이 사용 가능
```java
IntStream IntStream.of(int... values)
IntStream IntStream.of(int[])
IntStream Arrays.stream(int[])
IntStream Arrays.stream(int[] array, int start, int end)
```

### 2.4 특정 범위 정수
지정된 범위의 연속된 정수를 스트림으로 생성해서 반환
```java
IntStream IntStream.range(int begin, int end)
```

### 2.5 임의의 수
난수를 무한 스트림으로 생성해서 반환 (int, long, double 모두 같음)
- `doubles`만 `0.0 <= x < 1.0`이고 `나머지`는 `Integer`, `Long`의 최대, 최소 값이 범위이다.

```java
IntStream ints([[long streamSize,] [int begin, int end]]) // 스트림 크기, begin <= x < end 사이의 난수를 생성

//예시
IntStream intStream = new Random().ints();
intStream.limit(5).forEach(System.out::println);
```

### 2.6 람다식
람다식을 매개변수로 받아 람다식의 결과를 기반으로 무한 스트림을 생성
- 기본형 스트림 타입의 참조변수로 다룰 수 없다.
```java
static <T> Stream<T> iterate(T seed, UnaryOperator<T> f)
static <T> Stream<T> generate(Supplier<T> s)

//예시
Stream<Integer> evenStream = Stream.iterate(0, n -> n + 2);
Stream<Double> randomStream = Stream.generate(Math::random);
```

### 2.7 파일
`java.nio.file.Files`는 파일을 다루는데 필요한 유용한 메서드들을 제공한다.
- `list()`: 지정된 디렉토리에 있는 파일의 목록을 소스로 하는 스트림을 생성해서 반환
```java
Stream<Path> Files.list(Path dir)
```

### 2.8 빈 스트림
요소가 하나도 없는 비어있는 스트림을 생성해서 반환
- 스트림에 연산을 수행한 결과가 하나도 없을 때, null보다 빈 스트림을 반환하는 것이 낫다.
```java
//예시
Stream emptyStream = Stream.empty();
long count = emptyStream.count();
```

### 2.9 두 스트림 연결
- `Stream.concat()`: 두 스트림을 하나로 연결

```java
Stream<String> str1 = Stream.of(str1);
Stream<String> str2 = Stream.of(str2);
Stream<String> str3 = Stream.concat(str1, str2);
```
## 3. 스트림의 연산
스트림의 연산은 최종 연산이 수행되기 전까지는 중간 연산이 수행되지 않는다.

### 3.1 중간 연산

```java
//스트림의 원소들 간의 중복 제거
Stream<T> distinct() 
//조건에 안 맞는 요소 제외
Stream<T> filter(Predicate<? super T> predicate) // 하나의 스트림에 여러 번 사용 가능
//스트림의 일부를 잘라낸다.
Stream<T> limit(long maxSize) 
//스트림의 일부를 건너뛴다.
Stream<T> skip(long n) 
//스트림의 중간 연산 결과를 확인 (요소 소모 X)
Stream<T> peek(Consumer<T> action) 
//스트림의 요소를 정렬
Stream<T> sorted()
Stream<T> sorted(Comparator<? super T> comparator) 
//스트림의 요소를 변환
Stream<R>    map(Function<? super T, ? extends R> mapper) // 하나의 스트림에 여러 번 사용 가능
DoubleStream mapToDouble(ToDoubleFunction<? super T> mapper)
IntStream    mapToInt(ToIntFunction<? super T> mapper)
LongStream   mapToLong(ToLongFunction<? super T> mapper)

Stream<R>    flatMap(Function<T, Stream<R>> mapper)
DoubleStream flatMapToDouble(Function<T, DoubleStream> m)
IntStream    flatMapToInt(Function<T, IntStream> m)
LongStream   flatMapToLong(Function<T, LongStream> m)
```

#### a. sorted(Comparator<? super T> comparator)
- Comparator를 지정하지 않으면 스트림 요소의 기본 정렬 기준(Comparable)으로 정렬한다. (단 스트림의 요소는 Comparable을 구현해야만 한다.)

```java
//기본 정렬 
strStream.sorted();
strStream.sorted((s1, s2) -> s1.compareTo(s2)); // 정렬 기준 지정
strStream.sorted(String::compareTo); // 위와 동일

//역순 정렬
strStream.sorted(Comparator.reverseOrder());

//대소문자 구분 X
strStream.sorted(String.CASE_INSENSITIVE_ORDER); // 오름차순 정렬
strStream.sorted(String.CASE_INSENSITIVE_ORDER.reversed()); // 내림차순 정렬

//길이 순 정렬
strStream.sorted(Comparator.comparing(String::length))
strStream.sorted(Comparator.comparingInt(String::length))
strStream.sorted(Comparator.comparing(String::length).reversed())

//예시
studentStream.sorted(Comparator.comparingInt(Student::getBan)
                            .thenComparingInt(Student::getTotalScore)
                            .thenComparing(Student::getName)
                            .forEach(System.out::println))
```

#### b. map()
스트림의 요소에 저장된 값 중에서 원하는 필드만 뽑아내거나 특정 형태로 변환해야 할 때 사용

```java
//예시
fileStream.map(File::getName)
    .filter(s -> s.indexOf('.') != -1)
    .map(s -> s.substring(s.indexOf('.')+1)
    .map(String::toUpperCase)
    .distinct()
    .forEach(System.out::print));
```

#### c. mapToInt(), mapToLong(), mapToDouble()
객체 스트림을 기본형 스트림으로 변환해서 반환
- `Stream<Integer>`를 사용하는 것보다 `IntStream`을 사용하는 것이 더 효율적이다.
```java
//Stream<String> -> IntStream
mapToInt(Integer::parseInt)
//Stream<Integer> -> IntStream
mapToInt(Integer::intValue)
```

#### 기본형 스트림이 제공하는 최종 연산 메서드

```java
//IntStream의 경우
int count() //요소들의 개수를 반환
int sum()   //모든 요소들의 합을 반환
OptionalInt max() //최댓값을 OptionalInt 객체로 반환
OptionalInt min() //최솟값을 OptionalInt 객체로 반환
OptionalDouble average() //모든 요소들의 평균값을 OptionalDouble 객체로 반환

int reduce(int identity, IntBinaryOperator op) //스트림의 요소들에 대해 제공된 이진 연산자(op)를 순차적으로 적용하여 OptionalInt를 반환합니다.
OptionalInt reduce(IntBinaryOperator op)       //스트림의 요소들에 대해 제공된 항등값(identity)과 이진 연산자(op)를 순차적으로 적용하여 최종 결과값을 반환
 
boolean anyMatch()      //요소들 중 하나 이상이 주어진 조건을 만족하는지 여부를 반환
boolean allMatch()      //모든 요소가 주어진 조건을 만족하는지 여부를 반환
boolean noneMatch()     //요소들이 모두 주어진 조건을 만족하지 않는지 여부를 반환
OptionalInt findFirst() //첫 번째 요소를 OptionalInt 객체로 반환
OptionalInt findAny()   //임의의 요소를 OptionalInt 객체로 반환
```

#### summaryStatistics()
기본형 스트림이 제공하는 메서드를 한번에 여러번 사용하고 싶을 경우 사용한다.

```java
IntSummaryStatistics stat = scoreStream.summaryStatistics();
long totalCount = stat.getCount();
long totalScore = stat.getSum();
int    minScore = stat.getMin();
int    maxScore = stat.getMax();
double avgScore = stat.getAverage();
```

#### 반대로 기본형 스트림 -> 객체 스트림
```java
//IntStream -> Stream<T>
Stream<U> mapToObj(IntFunction<? extends U> mapper)

//IntStream -> Stream<Integer>
Stream<Integer> boxed()
```

#### 문자열 -> 기본형 스트림
`chars()`를 이용하면 String이나 StringBuffer에 저장된 문자들을 IntStream으로 다룰 수 있다.

```java
IntStream charStream = "12345".chars();
int charSum = charStream.map(ch -> ch - '0').sum();
```

#### d. flatMap()
스트림의 타입이 배열인 경우 `Stream<T>`로 변환해서 반환한다.
- `Stream<T[]> -> Stream<T>`
```java
//2차원 배열의 각각의 원소를 1차원 배열로 나열
Stream<String> strStrm = strArrayStrm.flatMap(Arrays::stream);

//1차원 배열에 문자열을 단어로 나눠서 나열
Stream<String> strStream = lineStream.flatMap(line -> Stream.of(line.split(" +")));
```

### 3.2 Optional<T> 와 OptionalInt
- `Optional<T>`는 T타입의 객체를 감싸는 래퍼 클래스이다.
    + 최종 연산의 결과를 Optional객체에 담아서 반환하면 담겨 있는 객체가 null이더라도 NullPointerException을 발생시키지 않는다.
- `OptionalInt`: 객체가 아닌 기본형 값을 갖는 Optional객체
#### a. Optional객체 생성
인자로 받아온 객체를 Optional객체에 담는다.

```java
//Optional 객체 생성
Optional<String> optVal = Optional.of("abc"); //인자가 null이면 NPE 발생
Optional<String> optVal = Optional.ofNullable(new String("abc")); //인자가 null이여서 NPE 발생 x

//Optional 객체 초기화 : null이 아닌 빈 객체로 초기화하는 것이 바람직하다.
Optional<String> optval = Optional.empty();
```

#### b. Optional객체의 값 가져오기
Optional객체에 저장된 값을 반환한다.
```java
T orElseGet(Supplier<? extends T> other)
T orElseThrow(Supplier<? extends X> eceptionSupplier)

//예시
Optional<String> optVal = Optional.of("abc");
String str1 = optVal.get();      // 저장된 값이 null이면 NoSuchElementException 발생
String str2 = optVal.orElse(""); // 저장된 값이 null이면 대체 값("")을 반환
String str3 = optVal.orElseGet(String::new); // 저장된 값이 null이면 람다식을 이용해 대체할 값을 지정
String str4 = optVal.orElseThrow(NullPointException::new); // 저장된 값이 null이면 람다식을 통해 지정된 예외를 발생시킨다.
```

#### c. Optional의 중간 연산
Optional객체에도 `filter()`, `map()`, `flatMap()`을 사용할 수 있다.

```java
//예시
//Integer.parseInt는 인자가 null이면 NPE, 숫자로된 문자열이 아니거나 정수 범위를 넘어가는 인자의 경우 NumberformatException 발생
int result = Optional.of("123")
                    .filter(x -> x.length() > 0)
                    .map(Integer::parseInt).orElse(-1);
```

- `isPresent()`: Optional객체의 값이 null이면 false, 아니면 true를 반환

```java
if (Optional.ofNullable(str).isPresent()) {
    System.out.println(str);
}
```
- `isPresent(Consumer<T> block)`: 값이 있으면 주어진 람다식을 실행, 없으면 아무일도 하지 않는다.
```java
Optional.ofNullable(str).ifPresent(System.out::println);
```

- OptionalInt에 저장된 값 꺼내기

```java
Optional<T>    T get()
OptionalInt    int getAsInt()
OptionalLong   Long getAsLong()
OptionalDouble Double getAsDouble()
```

- OptionalInt에서 0과 empty의 차이

```java
OptionalInt opt = OptionalInt.of(0);
OptionalInt opt = OptionalIint.empty();

System.out.println(opt.isPresent());  // 0: true / empty: false
System.out.println(opt.getAsInt());   // 0: 0    / empty: NoSuchElementException 발생
System.out.println(opt.equals(opt2)); // false

//단, Optional<T>의 경우 null을 저장하면 비어있는 것과 동일하다.
Optional<String> opt = Optional.ofNullable(null);
Optional<String> opt2 = Optional.empty();

System.out.println(opt.equals(opt2)); // true
```

### 3.3 최종 연산

```java
//각 요소에 지정된 작업 수행
void forEach(Consumer<? super T> action)
void forEachOrdered(Consumer<? super T> action)

//스트림의 요소의 개수 반환
long count()
//스트림의 최대값/최소값을 반환
Optional<T> max(Comparator<? super T> comparator)
Optional<T> min(Comparator<? super T> comparator)

//스트림의 요소 하나를 반환
Optional<T> findAny() // 아무거나 하나
Optional<T> findFirst() // 첫 번째 요소

//주어진 조건을 모든 요소가 만족시키는지 확인
boolean allMatch(Predicate<T> p) // 모두 만족하는지
boolean anyMatch(Predicate<T> p) // 하나라도 만족하는지
boolean noneMatch(Predicate<T> p) // 모두 만족하지 않는지

//스트림의 모든 요소를 배열로 반환
Object[] toArray()
A[]      toArray(IntFunction<A[]> generator)

//스트림의 요소를 하나씩 줄여가면서 계산
Optional<T> reduce(BinaryOperator<T> accumlator)
T reduce(T identity, BinaryOperator<T> accumulator)
U reduce(U identity, BiFunction<U,T,U> accumulator, BinaryOperator<U> combiner) 

//스트림의 요소를 수집
R collect(Collector<T,A,R> collector)
R collect(Supplier<R> supplier, BiConsumer<R,T> accumulator, BiConsumer<R,R> combiner)
```

#### a. reduce()
스트림의 요소를 줄여나가면서 연산을 수행하고 최종결과를 반환
- 초기 값이 있는 경우
    + 초기 값(identity)과 스트림의 첫 번째 요소로 연산
    + 스트림의 요소가 하나도 없으면 초기값 반환

- count, sum, max, min, average와 같이 통계와 관련된 메서드는 최종 연산 메서드를 사용하는 것보다 기본형 스트림으로 변환하거나, reduce()나 collect()를 사용해서 얻는 것이 데이터를 더 유연하게 다룰 수 있다.
- 하지만, 최종 연산을 사용하는 것이 더 직관적이다.

```java
Optional<T> reduce(BinaryOperator<T> accumlator)
T reduce(T identity, BinaryOperator<T> accumulator)
U reduce(U identity, BiFunction<U,T,U> accumulator, BinaryOperator<U> combiner) 

//예시
int count = intStream.reduce(0, (a,b) -> a + 1);
int sum   = intStream.reduce(0, (a,b) -> a + b);
int max   = intStream.reduce(Integer.MIN_VALUE, (a,b) -> a>b ? a:b);
int min   = intStream.reduce(Integer.MAX_VALUE, (a,b) -> a<b ? a:b);

//max와 min는 초기값이 필요없기 때문에 Optional<T>를 반환하는 매개변수 하나짜리 reduce()를 사용하는 것이 낫다.
//단, 해당 스트림의 타입이 IntStream인 경우 OptionalInt를 사용해야한다.
OptionalInt max = intStream.reduce((a,b) -> a>b ? a:b);
OptionalInt min = intStream.reduce((a,b) -> a<b ? a:b);

//Integer클래스의 static 메서드인 max, min 사용
OptionalInt max = intStream.reduce(Integer::max);
OptionalInt min = intStream.reduce(Integer::min);
```


#### b. collect()
스트림의 요소를 수집하는 최종 연산

```java
collect()  // 스트림의 최종 연산 메서드, 매개변수로 컬랙터 필요
Collector  // 어떻게 수집할 것인지에 대한 방법이 정의되어 있는 인터페이스
Collectors // static 메서드로 미리 작성된 컬렉터를 제공하는 클래스 

//스트림의 요소를 수집
R collect(Collector<T,A,R> collector) 
R collect(Supplier<R> supplier, BiConsumer<R,T> accumulator, BiConsumer<R,R> combiner)
```

#### 스트림을 컬렉션과 배열로 변환
- `toList()`, `toSet()`, `toMap()`, `toCollection()`, `toArray()`

```java
//toList() [toSet()도 똑같음.]
List<String> names = stuStream.map(Student::getName).collect(Collectors.toList());
//toCollection(특정 컬렉션를 지정)
ArrayList<String> list = names.stream().collect(Collectors.toCollection(ArrayList::new));
//toMap(key, value)
Map<String, Person> map = personStream.collect(Collectors.toMap(p->p.getId(), p->p));
//toArray(담을 배열) [매개변수를 지정하지 않으면 Object[]로 반환됨]
Student[] stuNames = studentStream.toArray(Student[]::new);
```

#### 통계 정보 얻기
- `counting()`, `summingInt()`, `averagingInt()`, `maxBy()`, `minBy()`
- 보통 최종 연산을 이용하기 보다는 `intStream`이나 `reduce()`를 통해 통계 정보를 얻는다.
- 하지만, `groupingBy()`를 사용할 때 유용하다.

```java
long count = stuStream.count();
long count = stuStream.collect(counting());

long totalScore = stuStream.mapToInt(Student::getScore).sum();
long totalScore = stuStream.collect(summingInt(Student::getScore));

OptionalInt topScore = stuStream.maptoInt(Student::getScore).max();
Optional<Student> topScore = stuStream.max(Comparator.comparingInt(Student::getScore));
Optional<Student> topScore = stuStream.collect(maxBy(Comparator.comparingInt(Student::getScore)));

IntSummaryStatistics stat = stuStream.mapToInt(Student::getScore).summaryStatistics();
IntSummaryStatistics stat = stuStream.collect(summarizingInt(Student::getScore));
```

#### reducing()
- IntStream에는 매개변수 3개 짜리 collect만 정의되어 있으므로 boxed()를 통해 Stream<Integer>로 변환해야 매개변수 1개 짜리를 사용할 수 있다.
```java
Collector reducing(BinaryOperator<T> op)
Collector reducing(T identity, BinaryOperator<T> op)
Collector reducing(U identity, Function<T,U> mapper, BinaryOperator<U> op)

//예시
IntStream intStream = new Random().ints(1, 46).distinct(),limit(6);

OptionalInt max = intStream.reduce(Integer::max);
OptionalInt max = intStream.boxed().collect(reducing(Integer::max));

long sum = intStream.reduce(0, (a,b) -> a + b);
long sum = intStream.boxed().collect(reducing(0, (a,b) -> a + b));

int grandTotal = stuStream.map(Student::getScore).reduce(0, Integer::sum);
int grandTotal = stuStream.collect(reducing(0, Student::getScore, Integer::sum));
```

#### joining()
문자열 스트림의 모든 요소를 하나의 문자열로 연결해서 반환 (구분자, 접두사, 접미사 지정 가능)

```java
String studentNames = stuStream.map(Student::getName).collect(joining());
String studentNames = stuStream.map(Student::getName).collect(joining(","));
String studentNames = stuStream.map(Stduent::getName).collect(joining(",", "[", "]"))
```

#### 그룹화 & 분할
- `groupingBy`(그룹화): 스트림의 요소를 특정 기준으로 그룹화하는 것의 의미
- `partitioningBy`(분할): 스트림의 요소를 두 가지, 지정된 조건에 일치하는 그룹과 일치하지 않는 그룹으로의 분할을 의미
- 스트림을 두 개의 그룹으로 나눌 경우 pratitioningBy()를 사용하고 나머지는 groupingBy()를 사용
- 그룹화와 분할은 `Map`에 담겨 반환된다.

```java
Collector groupingBy(Function classifier)
Collector groupingBy(Function classifier, Collector downstream)
Collector groupingBy(Function classifier, Supplier mapFactory, Collector downstream)

Collector partitioningBy(Predicate predicate)
Collector partitioningBy(Predicate predicate, Collector downstream)
```

#### partitioningBy()

```java
//예시
//1. 기본 분할
Map<Boolean, List<Student>> stuBySex = stuStream.collect(partitioningBy(Student::isMale));
List<Student> male = stuBySex.get(true);    // 남학생 목록
List<Student> female = stuBySex.get(false); // 여학생 목록

//2. 기본 분할 + 통계 정보
Map<Boolean, Long> stuNumBySex = stuStream.collect(partitioningBy(Student::isMale, counting()));
System.out.println(stuNumBySex.get(true));  // 남학생: 8
System.out.println(stuNumBySex.get(false)); // 여학생: 10

Map<Boolean, Optional<Student>> topScoreBySex = stuStream.collect(partitioningBy(Student::isMale, maxBy(comparingInt(Student::getScore))));
System.out.println(topScoreBySex.get(true));  // 남학생 1등: Optional[김철수, 남, 1, 1, 300]
System.out.println(topScoreBySex.get(false)); // 여학생 1등: Optional[김영희, 여, 1, 1, 250]

//3. Optional<Student>가 아닌 Student객체로 반환
Map<Boolean, Optional<Student>> toScoreBySex = stuStream.collect(
    partitioningBy(Student::isMale, 
    collectingAndThen(maxBy(comparingInt(Student::getScore)), Optional::get)
        )
    );
System.out.println(topScoreBySex.get(true));
System.out.println(topScoreBySex.get(false));

//4. 이중 분할
Map<Boolean, Map<Boolean, List<Student>>> failedStuBySex = stuStream.collect(
    partitioningBy(Student::isMale,
    partitioningBy(s -> s.getScore() < 150)
        )
    );
List<Student> failedMaleStu = failedStuBySex.get(true).get(true);    // 남학생 중 성적이 150점 미만인 리스트
List<Student> failedFemaleStu = failedStuBySex.get(false).get(true); // 여학생 중 성적이 150점 미만인 리스트
```

#### groupingBy()
그룹화를 하면 기본적으로 `List<T>`에 담아 반환한다.
```java
//예시
//1. 기본 그룹화
Map<Integer, List<Student>> stuByBan = stuStream.collect(groupingBy(Student::getBan)); // toList()가 생략된 상태
Map<Integer, List<Student>> stuByBan = stuStream.collect(groupingBy(Student::getBan, toList()));
//2. 다른 컬렉션에 담아서 반환
Map<Integer, HashSet<Student>> stuByBan = stuStream.collect(groupingBy(Student::getBan, toCollection(HashSet::new)));

//3. 그룹화 + 통계 정보
Map<Student.Level, Long> stuByLevel = stuStream
    .collect(groupingBy(s -> {
            if (s.getScore() >= 200)      return Student.Level.HIGH;
            else if (s.getScore() >= 100) return Student.Level.MID;
            else                          return Student.Level.LOW;
        }, counting())
    ); // [MID] - 8명, [HIGH] - 8명, [LOW] - 2명

//4. 다수준 그룹화
Map<Integer, Map<Integer, List<Student>>> stuByHakAndBan = stuStream
        .collect(groupingBy(Student::getHak,
                groupingBy(Student::getBan)
        )); // 학년 별로 그룹화 후 반 별로 그룹화

//5. collectingAndThen을 이용해 각 반의 1등 구하기
Map<Integer, Map<Integer, List<Student>>> topStuByHakAndBan = stuStream
        .collect(groupingBy(Student::getHak,
                groupingBy(Student::getBan,
                        collectingAndThen(
                            maxBy(comparingInt(Student::getScore)),
                            Optional::get
                        )
                )
        ));

//6. 그룹화한 후 mapping을 이용해서 원하는 컬렉션에 저장
Map<Integer, Map<Integer, Set<Student.Level>>> stuByHakAndBan = stuStream
    .collect(
        groupingBy(Student::getHak,
        groupingBy(Student::getBan,
            mapping(s -> {
                if (s.getScore() >= 200)      return Student.Level.HIGH;
                else if (s.getScore() >= 100) return Student.Level.MID;
                else                          return Student.Level.LOW;
                }, toSet())
            )
        )
    );
```

#### Collector 구현하기
```java
//Collector 인터페이스
public interface Collector<T, A, R> {
    Supplier<A>       supplier();    // 수집 결과를 저장할 공간 제공
    BiConsummer<A, T> accumulator(); // 스트림의 요소를 supplier에 의해 제공된 공간에 누적하는 방법 제공
    BinaryOperator<A> combiner();    // 병렬 스트림의 경우, 여러 쓰레드에 의해 처리된 결과를 합칠 방법 제공
    Function<A, R>    finisher();    // 결과를 최종적으로 변환할 방법 제공
    Set<Characteristics> characteristics(); // 컬렉션의 특성이 담긴 Set을 반환

    //...
}

//finisher()의 경우 변환이 필요없다면, 항등 함수를 반환한다.
public Function finisher() {
    return Function.identity(); // 항등 함수 (return x -> x;)
}
```
- `Set<Characteristics> characteristics()`: 컬렉터가 수행하는 작업의 속성 정보를 Set에 담아 제공

```java
//속성 정보
Characteristics.CONCURRENT      // 병렬로 처리할 수 있는 작업
Characteristics.UNORDERED       // 스트림의 요소의 순서가 유지될 필요가 없는 작업
Characteristics.IDENTITY_FINISH // finisher()가 항등 함수인 작업
```

```java
//예시
public Set<Characteristics> characteristics() {
    return Collections.unmodifiableSet(EnumSet.of(
                Collector.Characteristics.CONCURRENT,
                Collector.Characteristics.UNORDERED
            ));
}

//아무런 속성도 저장하고 싶지 않은 경우 비어있는 Set을 반환
Set<Characteristics> characteristics() {
    return Collections.emptySet();
}
```
## Reference
> 자바의 정석 - 남궁성

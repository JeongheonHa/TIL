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
Stream<R> map(Function<? super T, ? extends R> mapper) // 하나의 스트림에 여러 번 사용 가능
DoubleStream mapToDouble(ToDoubleFunction<? super T> mapper)
IntStream mapToInt(ToIntFunction<? super T> mapper)
LongStream mapToLong(ToLongFunction<? super T> mapper)

Stream<R> flatMap(Function<T, Stream<R>> mapper)
DoubleStream flatMapToDouble(Function<T, DoubleStream> m)
IntStream flatMapToInt(Function<T, IntStream> m)
LongStream flatMapToLong(Function<T, LongStream> m)
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

#### 기본형 스트림이 제공하는 메서드
최종 연산 메서드이므로 1번 밖에 사용하지 못한다.
```java
int            sum()        //스트림의 모든 요소의 총합
OptionalDouble average()    //스트림의 모든 요소의 평균
OptionalInt    max()        //스트림의 요소 중 가장 큰 값
OptionalInt    min()        //스트림의 요소 중 가장 작은 값
```

#### summaryStatistics()
기본형 스트림이 제공하는 메서드를 한번에 여러번 사용하고 싶을 경우 사용한다.

```java
IntSummaryStatistics stat = scoreStream.summaryStatistics();
long totalCount = stat.getCount();
long totalScore = stat.getSum();
double avgScore = stat.getAverage();
int    minScore = stat.getMin();
int    maxScore = stat.getMax();
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
`Optional<T>`는 T타입의 객체를 감싸는 래퍼 클래스이다.
- 최종 연산의 결과를 Optional객체에 담아서 반환하면 담겨 있는 객체가 null이더라도 NullPointerException을 발생시키지 않는다.

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

```java
//isPresent(): Optional객체의 값이 null이면 false, 아니면 true를 반환
if (Optional.ofNullable(str).isPresent()) {
    System.out.println(str);
}
//isPresent(Consumer<T> block): 값이 있으면 주어진 람다식을 실행, 없으면 아무일도 하지 않는다.
Optional.ofNullable(str).ifPresent(System.out::println);
```

#### d. Optional의 최종 연산
```java
Optional<T> findAny()
Optional<T> findFirst()
Optional<T> max(Comparator<? super T> comparator)
Optional<T> min(Comparator<? super T> comparator)
Optional<T> reduce(BinaryOperator<T> accumulator)
```

#### e. OptionalInt, OptionalLong, OptionalDouble 
- `OptionalInt`: 객체가 아닌 기본형 값을 갖는 Optional객체
- 나머지 `OptionalLong`, `OptionalDouble` 모두 똑같다.
```java
//IntStream에 정의된 메서드 - OptionalInt로 반환
OptionalInt    findAny()
OptionalInt    findFirst()
OptionalInt    reduce(IntBinaryOperator op)
OptionalInt    max()
OptionalInt    min()
OptionalDouble average()
```

```java
//OptionalInt에 저장된 값 꺼내기
Optional<T>    T get()
OptionalInt    int getAsInt()
OptionalLong   Long getAsLong()
OptionalDouble Double getAsDouble()
```

```java
//OptionalInt에서 0과 empty의 차이
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

## Reference
> 자바의 정석 - 남궁성

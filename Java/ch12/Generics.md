# Generics

## 1. 지네릭스
- 컴파일시 타입을 체크해 주는 기능
- 지네릭스의 장점
    + `타입 안정성`을 제공
    + `타입체크`와 `형변환을 생략`할 수 있으므로 코드가 간결해진다.
> - 실행 중 발생하는 에러를 컴파일 에러로 바꾸는 것이 기본 목적이다. </br>
> - 일반 클래스안에 Object타입을 포함하는 클래스는 지네릭 클래스로 바뀌었다.

## 2. 지네릭 클래스 선언
- Object타입 대신 `T`와 같은 `타입변수` 사용
- `참조변수`의 매개변수화된 타입과 `생성자`의 매개변수화된 `타입`은 반드시 `일치` (jdk1.7부터 생성자의 타입지정 생략가능 `<>`)
- 객체를 다시 만들때마다 다르게 줄 수 있다. (like 메서드의 매개변수)
- 컴파일 후 지네릭 클래스는 원시 타입으로 바뀐다.
> 무조건 "T"를 사용하는 것보다 상황에 맞는 타입 변수를 사용하도록 하자 !! </br> 
> 예를 들면 E,K,V

```java
// 일반 클래스
class Box {
    Object item;

    void setItem(Object item) { this.tiem = item; }
    Object getItem() { return item; }
}
// 지네릭 클래스
class Box<T> {
    T item;

    void setItem(T item) { this.item = item; }
    T getItem() { return item; }
}

// 지네릭 클래스 선언
Box<String> b = new Box<String>();  // 타입 T 대신, 실제 타입을 지정
b.setItem(new Object());    // error, String이외의 타입은 지정불가
b.setItem("ABC");           // Ok, String 타입이므로 가능
String item = b.getItem(); // getItem()의 반환타입이 String으로 바뀌었으므로 형변환할 필요 x

// Box<T> : 지네릭 클래스
// T : 타입 변수
// Box : 원시 타입
// <String> : 대입된 타입 or 매개변수화된 타입(parameterized type)
```

## 3. 지네릭 타입과 다형성
- 참조 변수와 생성자의 대입된 타입은 일치해야 한다.
```java
ArrayList<Tv> list = new ArrayList<Tv>();       // ok
ArrayList<Product> list = new ArrayList<Tv>();  // error
```

- 지네릭 클래스간의 다형성은 성립 (대입된 타입은 일치해야한다.)
```java
List<Tv> list = new ArrayList<Tv>();    // ok, ArrayList가 List구현(다형성)
List<Tv> kust = new LinkedList<Tv>();   // ok, LinkedList가 List구현(다형성)
```

- 매개변수의 다형성도 성립
```java
ArrayList<Product> list = new ArrayList<Product>();
list.add(new Product());
list.add(new Tv());
list.add(new Audio());

// add메서드의 정의
boolean add()(E e) { ... }
// 타입이 대입된 add메서드의 정의
boolean add(Product e) { ... }  // 매개변수의 다형성으로 인해 Product의 자손타입의 인스턴스도 매개변수로 받을 수 있다. 
```

## 4. Iterator<E>
```java
// 제네릭 Iterator인터페이스
public interface Iterator<E> {
    boolean hasNext();
    E next();
    void remove();
}

Iterator<Student> it = list.iterator();
while(it.hasNext()) {
    // Student s = it.next(); // 형변환 x
    // System.out.println(s.name);
    System.out.println(it.next().name); // 형변환을 안해도 되서 한줄로 간편히 가능 (위 두줄을 한 코드로 합한것)
}
```

## 5. HashMap<K,V>
- 여러 개의 타입 변수가 필요한 경우, `콤마(,)`를 구분자로 선언
- `K` : key, `V` : value
```java
HashMap<String, Student> map = new HashMap<String, Student>();  // 참조변수의 대입된 타입, 생성자의 대입된 타입 일치
map.put("하정헌", new Student("하정헌",1,1,100,100,100));

// HashMap 지네릭 클래스 정의
public class HashMap<K,V> extends AbstractMap<K,V> {
    ...
    public V get(Object key) { /* 내용생략 */ }
    public V put(K key, V value) { /* 내용생략 */ }
    public V remove(Object key) { /* 내용생략 */ }
    ...
}
```
- get(Object key)에서 Obect를 놓아둔 이유   
: hash()메서드의 매개변수로 Object타입을 받아오기 때문에 굳이 get()메서드의 매개변수 타입을 바꿔 줄 필요가 없기 때문이다 (불필요한 형변환 x)
```java
// get매서드의 정의
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
// hash()메서드의 정의
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

## 6. 제한된 지네릭 클래스
- `extends`로 대입할 수 있는 `타입을 제한`
```java
// Fruit의 자손만 타입으로 지정가능
class FruitBox<T extends Fruit> { 
    ArrayList<T> list = new ArrayList<T>();
    ...
}
FruitBox<Apple> appleBox = new FruitBox<Apple>();   // ok, apple은 Fruit의 자손
FruitBox<Toy> toyBox = new FruitBox<Toy>();         // error, Toy는 Fruit과 상속관계 x
```

- `인터페이스`인 경우에도 `extends`를 사용해서 `타입을 제한`(`주의` : implements 아님)
```java
class Fruit implements Eatable {}
interface Eatable {}
class FruitBox<T extends Fruit & Eatable> {...} // 인터페이스를 같이 제한할 경우 `&`를 사용 (콤마(,)아님)
                                                
// Fruit이 Eatable인터페이스를 구현했으므로 Fruit클래스 하나만 제한해도 된다. (&도 가능)
```

## 7. 지네릭스의 제약
- `static멤버`에 타입 변수 `사용 불가` (1)    
: 타입 변수에 대입은 인스턴스 별로 다르게 가능하지만 `static멤버`는 인스턴스에 `공통으로 존재`하기 때문에 사용할 수 없다.
> T는 인스턴스 변수로 간주된다.

```java
Box<Apple> appleBox = new Box<Apple>();
Box<Grape> grapeBox = new Box<Grape>();

class Box<T> {
    static T item;  // error
    static int compare(T t1, T t2) {...}    // error
}
```

- `객체나 배열을 생성`할 때 타입 변수 `사용 불가` (타입 변수로 배열 선언은 가능) (2)    
: new 연산자는 컴파일 시점에 뒤에 타입이 확정되어야만 하지만 타입변수는 확정되지 않았기 때문에 타입변수를 사용할 수 없다.

```java
class Box<T> {
    // 타입변수로 배열 선언
    T[] itemArr;    // 
    // 배열 생성에 타입변수 사용
    T[] toArray() {
        T[] tmpArr = new T[itemArr.length]; // error, `new T[], new T -> x`
    }
}
```
> - 같은 이유로 instanceof연산자도 타입 변수와 사용 불가 </br>
> - 지네릭 배열을 생성해야만 한다면 Reflection API의 newInstance()와 같이 `동적으로 객체를 생성하는 메서드로 배열을 생성`하거나, </br> 
`Object배열을 생성해서 복사한 다음 "T[]"로 형변환`하는 방법을 사용하자 !!

## 8. 와일드 카드 <?>
- `하나의 참조 변수`로 `대입된 타입이 다른 객체`를 `참조` 가능
```java
ArrayList<? extends Product> list = new ArrayList<Tv>();    // ok
list = new ArrayList<Audio>(); // ok
```
- 형태
    + `<? extends T>` : 와일드 카드의 상한 제한. T와 그 자손들만 가능
    + `<? super T>` : 와일드 카드의 하한 제한. T와 그 조상들만 가능
    + `<?>` : 제한없음. 모든 타입 가능. <? extends Object>와 동일

- 메서드의 매개변수에 와일드 카드 사용

```java
// 메서드 정의
static Juice makeJuice(FruitBox<? extends Fruit> box) {
    Stirng tmp = "";
    ...
}
System.out.println(Juicer.makeJuice(new FruitBox<Fruit>()));    // ok
System.out.println(Juicer.makeJuice(new FruitBox<Apple>()));    // ok, Apple이 Fruit클래스의 자손
```

## 9. 지네릭 메서드
- 지네릭 타입이 선언된 메서드(타입 변수는 메서드 내에서만 유효)
- 클래스의 타입 매개변수 <T>와 메서드의 타입 매개변수 <T>는 별개
- 반환타입 앞에 선언
```java
class FruitBox<T> {
    ...
    static <T> void sort(List<T> list, Comparator<? super T> c) {
        ...
    }
}
```
> `주의` : 클래스와 메서드의 타입문자가 일치하지만 but `다른 타입 변수`이다. (like iv vs lv)

- 메서드를 `호출할 때`마다 타입을 `대입`해야한다 (`대부분 생략 가능`)

```java
FruitBox<Fruit> fruitBox = new FruitBox<Fruit>();
FruitBox<Apple> appleBox = new FruitBox<Apple>();
...
// 생략 안한 경우
System.out.println(Juicer.<Fruit>makeJuice(fruitBox));
System.out.println(Juicer.<Apple>makeJuice(appleBox));
// 생략한 경우
System.out.println(Juicer.makeJuice(fruitBox))
System.out.println(Juicer.makeJuice(appleBox));
// 메서드 정의
static <T extends Fruit> Juice makeJuice(FruitBox<T> box) {
    String tmp = "";
    for(Fruit f : box.getList()) tmp += f + " ";
    return new Juice(tmp);
}
```

- 메서드를 호출할 때 `타입을 생략하지 않을 때`는 `클래스 이름 생략 불가` (드물다)

```java
System.out.println(<Fruit>makeJuice(fruitBox));         // error
System.out.println(this.<Fruit>makeJuice(fruitBox));    // ok
System.out.println(Juicer.<Fruit>makeJuice(fruitBox));  // ok

```

## 10. 지네릭 메서드 vs 와일드 카드가 사용된 메서드
- `와일드 카드`는 `하나의 참조변수`로 `서로 다른 타입이 대입된 여러 지네릭 객체`를 다루기 위한 것이고 </br>
`지네릭 메서드`는 `메서드를 호출할 때`마다 `다른 지네릭 타입을 대입`할 수 있게 하는 것이 목적이다.
```java
// 지네릭 메서드 정의
static <T extends Fruit> Juice makeJuice(FruitBox<T> box) {
    String tmp = "";
    for(Fruit f : box.getList()) tmp += f + " ";
    return new Juice(tmp);
}

// 와일드 카드가 사용된 메서드 정의
static Juice makeJuice(FruitBox<? extends Fruit> box) {
    String tmp = "";
    for(Fruit f : box.getList()) tmp += f + " ";
    return new Juice(tmp);
}
```
> 와일드 카드로 안될 때 지네릭 메서드를 사용하는 경우가 많다.

## 11. 지네릭 타입의 형변환
- `지네릭 타입`과 `원시 타입`간의 `형변환`은 바람직 하지 않다(경고 발생)
```java
Box<Object> objOBx = null;
// 지네릭 타입 -> 원시 타입 (경고 발생)
Box box = (Box)objBox;
// 원시 타입 -> 지네릭 타입 (경고 발생)
objBox = (Box<Object>)box;
```

- 와일드 카드가 사용된 지네릭 타입으로는 형변환 가능
```java
Box<Object> objBox = (Box<Object>)new Box<String>();    // error, 형변환 x
Box<? extends Object> wBox = (Box<? extends Object>)new Box<String>();  // ok
Box<? extends Object> wBox = new Box<String>(); // ok, 형변환 생략
```
- 반대의 경우
```java
class Apple extends Fruit { public String toString() { return "Apple"; }}
class Ex {
    public static void main(String[] args) {
        // FruitBox<Apple> -> FruitBox<? extends Fruit> 
        FruitBox<? extends Fruit> abox = new FruitBox<Apple>(); // 가능
        // FruitBox<? extends Fruit> -> FruitBox<Apple> 
        FruitBox<Apple> appleBox = (FruitBox<Apple>)abox;   // 가능, 미확인 타입으로 형변환 경고 발생
    }
}
class FruitBox<T extends Fruit & Eatable> extends Box<T> {}
class Box<T> { }
```

- `<? extends Object>`를 줄여서 `<?>`로 쓸 수 있다.
```java
Optional<?> EMPTY = new Optional<?>();      // error, new연산자는 컴파일 단계에서 타입을 확정해야한다.
Optional<?> EMPTY = new Optional<Object>(); // ok
Optional<?> EMPTY = new Optional<>();       // ok

// <?>
class Box<T extends Fruit> {}
Box<?> b= new Box<>;    // Box<?> b = new Box<Fruit>;이다.
```


## 12. 지네릭 타입의 제거
- 이전 버전과의 `하위 호환성`과 `안정성` 때문에 지네릭 타입은 `컴파일 타임에만 존재`하고 컴파일 후에는 적절한 타입으로 바뀐다.
- 컴파일러는 지네릭 타입을 제거하고 필요한 곳에 형변환을 넣는다.
- 제거 과정
    + 지네릭 타입의 `경계`(bound)를 `제거`
    + 지네릭 타입 제거 후에 타입이 일치하지 않으면, `형변환을 추가`
    + `와일드 카드`가 포함된 경우, 적절한 타입으로 `형변환 추가`
    
# Reference
> 자바의 정석 - 남궁성

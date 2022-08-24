# HashSet & TreeSet

## 1. HashSet
- Set인터페이스를 구현한 대표적인 컬렉션 클래스
- 순서를 유지하고싶은 경우 `LinkedHashSet`클래스를 사용하면 된다.
> 순서가 필요없고 중복이 없는 Set이 필요한 경우 HashSet을 사용하도록 하자.

### 1.1 HashSet의 생성자와 메서드
- 생성자

| 생성자 | 설명 |
|---|---|
| HashSet() | HashSet객체 생성 |
| HashSet(Collection c) | 주어진 컬렉션을 포함하는 HashSet객체를 생성 |
| HashSet(int initialCapacity) | 주어진 값을 초기용량으로하는 HashSet객체를 생성 |
| HashSet(int initialCapacity, float loadFactor) | 초기용량과 load factor를 지정하는 생성자 |
> `load Factor` : 컬렉션 클래스에 저장공간이 가득 차기 전에 미리 용량을 확보하기 위한 것으로 </br>
> 이 값을 0.8로 지정하면, 저장공간의 80%가 채워졌을 때 용량이 두 배로 늘어난다. `기본값은 0.75`이다

- 메서드

| 메서드 | 설명 |
|---|---|
| boolean add(Object o) </br> boolean addAll(Collection c) | 주어진 객체(o) 또는 Collection(c)의 객체들을 `추가` <br> 이미 저장되어 있는 요소와 중복된 요소를 추가할 경우 false 반환 |
| void clear() | 저장된 `모든 객체`를 `삭제` |
| boolean contains(Object o)</br> boolean containsAll(Collection c) | 주어진 객체(o) 또는 Collection(c)의 객체들이 포함되어 있는지 `확인` |
| boolean isEmpty() | HashSet이 비어있는지 `확인` |
| Iterator iterator() | Iterator를 `반환` |
| boolean remove(Object o) | 주어진 객체를 HashSet에서 `삭제` |
| boolean removeAll(Collection c) | 주어진 Collection에 포함된 객체들을 HashSet에서 `삭제` |
| boolean retainAll(Collection c) | 주어진 Collection에 포함된 객체만을 남기고 다른 객체들은 `삭제` 후 </br> 이 작업이 변화가 있으먼 `true` 변화가 없으면 `false` |
| int size() | 저장된 `객체의 개수`를 `반환` |
| Object[] toArray() | 저장된 객체를 `객체배열`로 `반환` |
| Object[] toArray(Object[] a) | 저장된 객체들을 주어진 `객체배열`(a)에 담아서 `반환` |
| Object clone() | HashSet을 `복제`해서 `반환`(얕은 복사) |

### 1.2 중복을 걸러내는 과정
- `add()메서드`는 새로운 요소를 추가하기 전에 기존에 저장된 요소와 같은 것인지 판별하기 위해 </br> 
추가하려는 요소의 `equals()`와 `hashCode()`를 `호출`하기 때문에 equals()와 hashCode()를 목적에 맞게 `오버라이딩` 해야한다.
- HashSet은 객체를 저장하기 전에 먼저 객체의 hashCode()메소드를 호출해서 해시 코드를 얻어낸 다음 저장되어 있는 객체들의 해시 코드와 비교한 뒤 같은 해시 코드가 있다면 다시 equals() 메소드로 두 객체를 비교해서 true가 나오면 동일한 객체로 판단하고 중복 저장을 하지 않습니다.    
문자열을 HashSet에 저장할 경우, 같은 문자열을 갖는 String객체는 동일한 객체로 간주되고 다른 문자열을 갖는 String객체는 다른 객체로 간주되는데, 그 이유는 String클래스가 hashCode()와 equals() 메소드를 재정의해서 같은 문자열일 경우 hashCode()의 리턴 값을 같게, equals()의 리턴 값은 true가 나오도록 했기 때문입니다.


```java
// 예제

// HashSet인스턴스 생성
HashSet set = new HashSet();
// Person인스턴스를 생성후 생성된 객체를 인자로 add()호출
// 생성된 객체의 hashCode()를 호출해 HashSet객체들의 hashCode와 비교 후 없으면 저장, 있으면 equals()호출
set.add(new Person("David",10));    
set.add(new Person("David",10));

class Person {
    // 인스턴스 변수 선언
    String name;
    int age;
    // 생성자
    Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    // equals()오버라이딩
    public boolean equals(Object obj) { // 다형성을 이용해서 모든 객체를 매개변수로 받아올 수 있다.
        // Object타입의 obj참조변수가 Person으로 형변환이 가능하다면
        if(obj instanceof Person) {
            // Person인스턴스의 변수에 접근하기위해 Person으로 형변환
            Person p = (Person)obj;
            // 나 자신의 name과 age를 p와 비교
            return this.name.equals(p.name) && this.age == p.age;
        }
        return false;
    }
    // hashCode()오버라이딩
    public int hashCode() {
        // 자신의 인스턴스변수의 hashCode를 반환
        return Objects.hashCode(name,age);  // int hashCode(Object... values)
    }
}
```

### 1.3 오버라이딩을 통해 작성된 hashCode()의 3가지 조건
- 실행 중인 애플리케이션 내의 동일한 객체에 대해서 여러 번 hashCode()를 호출해도 동일한 int값을 반환해야한다. </br> 
하지만, 실행시마다 동일한 int값을 반환할 필요는 없다. (단, equals메서드의 구현에 사용된 멤버변수의 값이 바뀌지 않았다고 가정)
- equals메서드를 이용한 비교에 의해서 true를 얻은 두 객체에 대해 각각 hashCode()를 호출해서 얻은 결과는 반드시 같아야한다.
- equals메서드를 호출했을 때 false를 반환하는 두 객체는 hashCode()호출에 대해 같은 int값을 반환하는 경우가 있어도 괜찮지만, </br> 
해싱(hashing)을 사용하는 컬렉션의 성능을 향샹시키기 위해서는 다른 int값을 반환하는 것이 좋다.

## 2. TreeSet
- `이진 탐색 트리`라는 데이터의 형태로 데이터를 저장하는 `컬렉션클래스`
- `정렬`, `범위검색`에 유리하다.
- TreeSet은 이진 탐색 트리의 성능을 향상시킨 `레드-블랙 트리`로 구현되어 있다.

### 2.1 이진 트리
- 링크드리스트처럼 여러 개의 노드가 서로 연결된 구조로 각 노드에 `최대 2개`의 노드를 연결할 수 있으며 `루트`라고 불리는 하나의 노드에서부터 시작해서 계속 확장해나간다.
- 위 아래로 연결된 노드는 `부모-자식 관계`에 있다고 한다.

```java
class TreeNode {
    // 왼쪽 자식 노드
    TreeNode left;
    // 객체를 저장하기 위한 참조 변수
    Object element;
    // 오른쪽 자식 노드
    TreeNode right;
}
```

### 2.2 이진 탐색 트리
- `왼쪽` : 부모 노드보다 `작은 값` , `오른쪽` : 부모 노드보다 `큰 값`
- `정렬`, `검색`, `범위검색`에 유리한 자료구조
- 데이터가 많아질수록 노드의 `추가/삭제`에 `시간이 많이 걸린다`. (비교 횟수가 증가하기 때문에)
- 중복된 값 저장 x (add()호출할 때 equals(), hashCode()를 호출해 비교)

### 2.3 TreeSet의 생성자와 메서드
- 생성자

| 생성자 | 설명 |
|---|---|
| TreeSet() | 기본 생성자 |
| TreeSet(Collection c) | 주어진 컬렉션을 저장하는 TreeSet을 생성 |
| TreeSet(Comparator comp) | 주어진 정렬조건으로 정렬하는 TreeSet을 생성 |
| TreeSet(SortedSet s) | 주어진 SortedSet을 구현한 컬렉션을 저장하는 TreeSet을 생성 |

- 메서드 1 (Collection 인터페이스)

| 메서드 | 설명 |
|---|---|
| boolean add(Object o) </br> boolean addAll(Collection c) | 주어진 객체(o) 또는 Collection(c)의 객체들을 `추가` <br> `이미 저장되어 있는 요소와 중복된 요소를 추가할 경우 false 반환` |
| void clear() | 저장된 `모든 객체` `삭제` |
| boolean contains(Object o) </br> boolean containsAll(Collection c) | 지정된 객체(o) 또는 Collection의 객체들이 포함되어 있는지 `확인` |
| boolean isEmpty() | TreeSet이 비어있는지 `확인` |
| Iterator iterator() | TreeSet의 iterator를 `반환` |
| boolean remove(Object o) | 지정된 객체 `삭제` |
| boolean retinAll(Collection c) | 주어진 컬렉션과 공통된 요소만을 남기고 `삭제` |
| int size() | 저장된 객체의 `개수` `반환` |
| Object[] toArray() | 저장된 객체를 `객체배열`로 `반환` |
| Object[] toArray(Object[] a) | 저장된 객체를 주어진 `객체배열`에 저장하여 `반환 `|

- 메서드 2 (TreeSet클래스)

| 메서드 | 설명 |
|---|---|
| Object ceiling(Object o) | 지정된 객체와 `같은 객체`를 `반환`, 없으면 `큰 값`을 가진 객체 중 제일 `가까운 값`의 객체를 `반환` (없으면 null) |
| Object clone() | TreeSet을 `복제`하여 `반환` |
| Comparator comparator() | TreeSet의 `정렬기준`을 `반환` |
| NavigableSet descendingSet() | TreeSet에 저장된 요소들을 `역순`으로 `정렬`해서 `반환` |
| Object first() | 정렬된 순서에서 `첫 번째 객체`를 `반환` |
| Object floor(Object o) | 지정된 객체와 `같은 객체`를 `반환`, 없으면 `작은 값`을 가진 객체 중 제일 `가까운 값`의 객체를 `반환` (없으면 null) |
| SortedSet headSet(Object toElement) | 지정된 객체보다 `작은 값`의 객체들을 `반환` |
| NavigableSet headSet(Object toElement, boolean inclusive) | 지정된 객체보다 `작은 값`의 객체들을 `반환` </br> inclusive가 true이면, 같은 값의 객체도 포함 |
| Object higher(Object o) | 지정된 객체보다 `큰 값`을 가진 객체 중 제일 `가까운 값`의 객체를 `반환`, (없으면 null)
| Object last() | 정렬된 순서에서 `마지막 객체`를 `반환` |
| Object lower(Object o) | 지정된 객체보다 `작은 값`을 가진 객체 중 제일 `가까운 값`의 객체를 `반환`, (없으면 null) |
| Object pollFirst() | TreeSet의 `첫 번째 요소`(제일 작은 값의 객체)를 `반환` |
| Object polllast() | TreeSet의 `마지막 요소`(제일 큰 값의 객체)를 `반환` |
| Spliterator spliterator() | TreeSet의 spliterator `반환` |
| SortedSet subSet(Object fromElement, Object toElement) | 범위 검색의 결과 `반환` (마지막 미포함) |
| NavigableSet<E> subSet(E fromElement, boolean fromInclusive, </br> E toElement, boolean toInclusive) | 범위 검색의 결과 `반환` </br> (fromInclusive가 true면 시작 값 포함, toInclusive가 true면 마지막 값 포함) |
| SortedSet tailSet(Object fromElement) | 지정된 객체보다 `큰 값`의 객체들을 `반환` |

### 2.4 TreeSet vs HashSet
- TreeSet은 정렬을 해주지 않아도 정렬이되고 HashSet은 정렬을 해주지 않으면 정렬이 되지 않는다.
- `TreeSet`은 `비교기준을 제공`해야하고 `HashSet`은 비교기준을 제공하지 않아도 된다.
- TreeSet에 저장되는 객체가 Comparable을 구현하던지, TreeSet에게 Comparator를 제공해서 두 객체를 비교할 방법을 알려줘야한다.

```java
// HashSet을 정렬하는 경우
Set set = new HashSet();

List list = new LinkedList(set);    // LinkedList(Collection c)
Collections.sort(list);             // Collections.sort(List list)
System.out.println(list);
```
```java
// 예제
class Ex {
    public static void main(String[] args) {
        Set set = new TreeSet(new TestComp());    // 정렬 필요 없음
        // Set set = new HashSet(); // 정렬 필요

        for(int i = 0; set.size() < 6; i++) {
            int num = (int)(Math.random() * 45) + 1;

            // set.add(new Integer(num))   // Integer에 Comparable이 구현되어 있음
            set.add(new Test());        // set의 add()메서드는 비교하면서 저장하기 때문에 비교기준을 정해줘야한다.
            set.add(new Test());
            set.add(new Test());
            set.add(new Test());
        }
        System.out.println(set);
    }
}
// <1> 저장되는 객체에 Comparable 구현
class Test implements Comparable {
    public int compareTo(Object o) {
        return 0;
    }
} 
// <2> TreeSet에 Comparator를 제공
class TestComp implements Comparator {
    public int compare(Object o1, Object o2) {
        return 0;   // 0 -> 같은 객체라고 판단, -1,1 -> 같은 객체가 아니라고 판단. 
    }
}
```

### 2.5 트리순회
- 이진 트리의 모든 노드를 한번씩 읽는 것을 트리 순회하고 한다.
- `전위순회`(preorder) : 부모를 먼저 읽고 자식을 읽는 방식
- `후의순회`(postorder) : 자식을 먼저 읽고 부모를 읽는 방식
- `중위순회`(inorder) : 자식-부모-자식 순으로 읽는 방식 (오름차순 정렬)
- `레벨순회`(levelorder) : 위에서부터 순서대로 읽는 방식


# References
> - 자바의 정석 - 남궁성   
> - [차근차근 개발일기+일상:티스토리](https://crazykim2.tistory.com/474)

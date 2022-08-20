# collections framework

## 1. collections framework
- collection : 여러 객체(데이터)를 모아 놓은 것
- framework : 표준화, 정형화된 체계적인 프로그래밍 방식
- `collections framework` : collection을 다루기 위한 표준화된 프로그래밍 방식 
    + collection을 쉽고 편하게 다룰 수 있는 다양한 클래스 제공 (like `저장`, `삭제`, `검색`, `정렬`)
    + `java.util`패키지에 포함되어 있으며 `jdk1.2`부터 제공된다.
- collection class : 다수의 데이터를 저장할 수 있는 클래스 (Vector, ArrayList, HashSet ...)
> `주의` : collections framework을 사용할 때는 `import java.util.*`사용한다는 점 잊지말자 !!

## 2. 컬렉션 프레임웍의 핵심 `인터페이스`
- `List` : 순서가 있는 데이터의 집합, 데이터의 중복을 허용한다. (`순서: o, 중복: o`)
    + 구현 클래스 : ArrayList, LinkedList, Stack, Vector ...
- `Set` : 순서가 없는 데이터의 집합, 데이터의 중복을 허용하지 않는다. (`순서: x, 중복: x`)
    + 구현 클래스 : HashSet, TreeSet ...
- `Map` : 키(key)와 값(value)의 쌍으로 이루어진 데이터의 집합   
          순서는 유지되지 않고, 키는 중복을 허용하지 않고, 값은 중복을 허용한다. (`순서: x, 중복: 키(x), 값(o)`)
    + 구현 클래스 : HashMap, TreeMap, Hashtable, Properties ...
> List(`자손`)와 Set(`자손`)의 공통 부분을 뽑아서 Collection(`조상`) 인터페이스를 만든 것이다.

## 3. Collection 인터페이스의 메서드
| 메서드 | 설명 |
|---|---|
| boolean add(Object o) </br> boolean addAll(Collection c) | 지정된 객체(o) 또는 Collection(c)의 객체들을 Collection에 `추가` |
| void clear() | Collection의 `모든 객체`를 `삭제` |
| boolean contains(Object o)</br> boolean containsAll(Collection c) | 지정된 객체(o) 또는 Collection(c)의 객체들이 Collection에 포함되어 있는지 `확인` |
| boolean isEmpty() | Collection이 비어있는지 `확인` |
| Iterator iterator() | Collection이 Iterator를 얻어서 `반환` |
| boolean remove(Object o) | 지정된 객체를 `삭제` |
| boolean removeAll(Collection c) | 지정된 Collection에 포함된 객체들을 `삭제` |
| boolean retainAll(Collection c) | 지정된 Collection에 포함된 객체만을 남기고 다른 객체들은 Collection에서 `삭제` 후 </br> 이 작업이 변화가 있으먼 `true` 변화가 없으면 `false` |
| int size() | Collection에 저장된 `객체의 개수`를 `반환` |
| Object[] toArray() | Collection에 저장된 객체를 `객체배열`(Object[])로 `반환` |
| Object[] toArray(Object[] a) | 지정된 배열에 Collection의 객체를 `저장`해서 `반환` |
| boolean equals(Object o) | 동일한 Collection인지 `비교` (모든 객체에 존재)|
| int hashCode() | Collection의 `hash code`를 `반환` (모든 객체에 존재) |

## 4. List인터페이스의 메서드 
- List인터페이스는 `Vector`, `ArrayList`, `LinkedList`클래스가 `구현`한다.

| 메서드 | 설명 |
|---|---|
| void add(int index, Object element) </br> boolean addAll(int index, Collection c) | 지정된 위치에 객체 또는 컬렉션에 포함된 객체들을 `추가` |
| Object get(int index) | 지정된 위치에 있는 객체를 `반환` |
| int indexOf(Object o) | 지정된 객체의 위치를 `반환` (`-->`) |
| int lastIndexOf(Object o) | 지정된 객체의 위치를 `반환` (`<--`) |
| ListIterator listIterator() </br> ListIterator listIterator(int index) | List의 객체에 접근할 수 있는 ListIterator를 `반환` |
| Object remove(int index) | 지정된 위치에 있는 객체를 `삭제`하고 삭제된 객체를 `반환` |
| Object set(int index, Object element) | 지정된 위치에 객체를 `저장` |
| void sort(Comparator c) | 지정된 비교자로 List를 `정렬` |
| List subList(int fromIndex, int toIndex) | 지정된 범위에 있는 객체를 `반환` |
> ArrayList인터페이스는 Vector인터페이스를 개선한 것으로 ArrayList는 동기화가 안되어 있으나 Vector는 자체적으로 동기화 되어있다.

## 5. Set인터페이스의 메서드
- Set인터페이스는 `HashSet`, SortedSet, `TreeSet`클래스가 `구현`했으며 TreeSet은 SortedSet의 `자손`이다.
- Set인터페이스의 메서드는 Collection인터페이스의 메서드와 동일하다.
- 집합과 관련된 메서드 (Collection에 변화가 있으면 true, 없으면 false 반환)

| 메서드 | 설명 |
|---|---|
| boolean addAll(Collection c) | 지정된 객체(o) 또는 Collection(c)의 객체들을 Collection에 `추가` (`합집합`)|
| boolean containsAll(Collection c) | 지정된 객체(o) 또는 Collection(c)의 객체들이 Collection에 포함되어 있는지 `확인` (`부분집합`) |
| boolean removeAll(Collection c) | 지정된 Collection에 포함된 객체들을 `삭제` (`차집합`) |
| boolean retainAll(Collection c) | 지정된 Collection에 포함된 객체만을 남기고 다른 객체들은 Collection에서 `삭제` 후 </br> 이 작업이 변화가 있으먼 `true` 변화가 없으면 `false` (`교집합`) |
> 이것 또한 Collection인터페이스의 메서드에 있는 것이다.

## 6. Map인터페이스의 메서드
- Map인터페이스는 Hashtable, `HashMap`, SortedMap클래스가 `구현`했으며    
  LinkedHashMap은 HashMap의 `자손`이고 `TreeMap`은 SortedMap의 `자손`이다.
- LinkedHashMap은 순서가 없는 Map을 보완하기위한 클래스로 `순서`가 있다.

| 메서드 | 설명 |
|---|---|
| void clear() | Map의 `모든 객체`를 `삭제` |
| boolean containKey(Object key) | 지정된 key객체와 일치하는 Map의 `key`객체가 있는지 `확인` |
| boolean containsValue(Object value) | 지정된 value개체와 일치하는 Map의 `value`객체가 있는지 `확인` |
| Object get(Object key) | 지정한 key객체에 대응하는 `value`객체를 찾아서 `반환` |
| boolean isEmpty() | Map이 비어있는지 `확인` |
| Object put(Object jey, Object value) | Map에 value객체를 key객체에 연결(`mapping`)하여 `저장` |
| void putAll(Map t) | 지정된 Map의 모든 `key-value쌍`을 `추가` |
| Object remove(Object key) | 지정된 key객체와 일치하는 key-value객체를 `삭제` |
| int size() | Map에 저장된 key-value쌍의 `개수`를 `반환` |
| Set entrySet() | Map에 저장되어 있는 `key-value`쌍을 Map.Entry타입의 객체로 저장한 Set을 반환 (`읽기`) |
| Set KeySet() | Map에 저장된 모든 `key`객체를 반환 (`읽기`) |
| Collection values() | Map에 저장된 모든 `value`객체를 반환 (`읽기`) |
| boolean equals(Object o) | 동일한 Map인지 `비교` |
| int hashCode() | `hash code`를 `반환` |
> - HashMap은 Hashtable을 개선한 것으로 HashMap은 동기화가 안되어 있으나 Hashtable은 동기화 되어있다.   
> - Collection values()에서 Collection은 list or Set 어떤 타입으로 와도 상관없기 때문에 Collection을 사용한다.     
> - entry : key-value쌍을 entry라고 한다.

# Reference
> 자바의 정석 - 남궁성

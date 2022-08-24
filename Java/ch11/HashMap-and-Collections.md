# HashMap & Collections

## 1. HashMap
- Map인터페이스를 구현한 대표적인 컬렉션 클래스
- Map인터페이스를 구현하여 데이터를 `키(key`)와 `값(value)`의 쌍으로 저장
- `해싱(hashing)기법`으로 데이터를 저장하기 때문에 데이터가 많아도 `검색이 빠르다`.
- HashMap(동기화 x)은 Hashtable(동기화 o) 의 신버전
- 순서 유지가 필요한 경우 `LinkedHashMap`을 사용한다.
> - key와 value로 나타내야할경우 HashMap을 사용하자 !!</br> 
> - Hashtable은 키와 값으로 null을 허용하지 않지만 HashMap은 허용한다. (map.put(null,null) 가능)

```java
// 정의
public class HashMap extends AbstractMap implements Map, Cloneable, Serializable {
    // 키와 값을 하나로 묶은 것을 저장하는 배열
    transient Entry[] table;
    ...
    static class Entry implements Map.Entry {
        final Object key;
        Object value;
        ...
    }
}

// Map.Entry는 Map인터페이스에 정의된 static inner interface이다
```
### 1.1 HashMap의 생성자와 메서드

- 생성자

| 생성자 | 설명 |
|---|---|
| HashMap() | HashMap객체 생성 |
| HashMap(int initialCapacity) | 지정된 값을 초기용량으로 하는 HashMap객체 생성 |
| HashMap(int initialCapacity, float loadFactor) | 지정된 초기용량과 load factor의 HashMap객체 생성 |
| hashMap(Map m) | 지정된 Map의 모든 요소를 포함하는 HashMap 생성 |

- 메서드 1 (Map인터페이스)

| 메서드 | 설명 |
|---|---|
| boolean containsKey(Object key) | HashMap에 지정된 `키`가 포함되어있는지 `확인` |
| boolean containsValue(Object value) | HashMap에 지정된 `값`이 포함되어있는지 `확인` |
| Object get(Object key) | 지정한 key객체에 대응하는 `value`객체를 찾아서 `반환` |
| Object put(Object key, Object value) | Map에 value객체를 key객체에 연결(`mapping`)하여 `저장` |
| void putAll(Map t) | 지정된 Map의 모든 `key-value쌍`을 `추가` |
| Object remove(Object key) | 지정된 key객체와 일치하는 key-value객체를 `삭제` |
| Set entrySet() | HashMap에 저장되어 있는 키와 값을 `entry`형태로 Set에 저장해서 `반환` (`읽기`) |
| Set keySet() | hashMap에 저장된 모든 `key`를 Set에 저장해서 `반환` (`읽기`) |
| Collection values() | HashMap에 저장된 모든 `value`객체를 Collection형태로 `반환` (`읽기`) |
| void clear() | HashMap의 `모든 객체`를 `삭제` |
| boolean isEmpty() | HashMap이 비어있는지 `확인` |
| int size() | HashMap에 저장된 요소의 `개수`를 `반환` |

- 메서드 2 (HashMap클래스)

| 메서드 | 설명 |
|---|---|
| Object clone() | 현재 HashMap을 `복제`해서 `반환` |
| Object getOrDefault(Object key, Object defaultValue) | 지정된 키의 값(객체)을 `반환` </br> 키를 못찾으면 기본값(defaultValue)으로 지정된 객체를 `반환` |
| Object replace(Object key, Object value) | 지정된 키의 값을 지정된 객체로 `대체` |
| boolean repalce(Object key, Object oldValue, Object newValue) | 지정된 키와 객체(oldValue)가 모두 일치하는 경우에만 </br> 새로운 객체(newValue)로 `대체` |

## 2. TreeMap
- TreeSet이 TreeMap을 따라서 만든 것이기 때문에 TreeSet과 TreeMap은 특성이 같다.
- `범위 검색`과 `정렬`에 유리한 컬렉션 클래스
- HashMap보다 데이터 추가/삭제에 시간이 더 걸린다.
- `검색`의 성능은 `HashMap`이 TreeMap보다 더 뛰어나고 `범위검색`이나 `정렬`이 필요한 경우 `TreeMap`을 사용하도록하자

## 3. 해싱(hashing)
- `해시함수`를 이용해서 `해시테이블`에 데이터를 `저장`, `검색`하는 방식
- `해시함수` : `키`를 넣으면 `저장위치`(hashcode)를 알려주는 함수
- `저장위치` : 배열의 index
- `해시테이블` : 배열과 링크드 리스트가 조합된 형태의 저장 공간 (like 2D 배열)
    + 배열의 장점인 `접근성`과 링크드 리스트의 장점인 `변경에 유리`하다는 점을 합했다.

### 3.1 해시테이블에 저장된 데이터를 가져오는 과정
- 키로 해시함수를 호출해서 해시코드를 얻는다. (1)
- 해시코드(해시함수의 반환값)에 대응하는 링크드 리스트를 배열에서 찾는다. (2)
- 링크드 리스트에서 키와 일치하는 데이터를 찾는다. (3)
> - 해시함수는 같은 키에 대해 항상 같은 해시코드를 반환해야한다.   
> - 서로 다른 키라도 같은 값의 해시코드를 반환할 수 있다.

## 4. Properties
- Hashtable을 상속받아 구현한 컬렉션 클래스
- Hashtable은 `키`와 `값`을 (Object, Object)의 형태로 저장하는데 비해 `Properties`는 `(String, String)`의 형태로 저장
- 주로 애플리케이션의 환경설정과 관련된 속성(properties)을 저장하는데 사용
- 데이터를 `파일`로부터 읽고 쓰는 편리한 기능을 제공

### 4.1 Properties의 생성자와 매서드
- 생성자

| 생성자 | 설명 |
|---|---|
| Properties() | Properties객체 생성 |
| Properties(Properties defaults) | 지정된 Properties에 저장된 목록을 가진 Properties객체 생성 |

- 메서드

| 메서드 | 설명 |
|---|---|
| String getProperty(String key) | 지정된 키의 `값`(value)을 `반환` |
| String getProperty(String key, String defaultValue) | 지정된 키의 `값`(value)을 `반환`, 키를 못찾으면 `defaultValue`를 `반환` |
| void list(PrintStream out) | 지정된 PrintStream에 저장된 `목록`을 `출력` |
| void list(PrintWriter out) | 지정된 PrintWriter에 저장된 `목록`을 `출력` |
| void load(InputStream inStream) | 지정된 InputStream으로부터 `목록`을 읽어서 `저장` |
| void load(Reader reader) | 지정된 Reader로부터 `목록`을 읽어서 `저장` |
| void loadFromXML(InputStream in) | 지정된 InputStream으로부터 XML문서를 읽어서, XML문서에 저장된 목록을 읽어다 담는다. (`load` & `store`) |
| Enumeration propertyNames() | 목록의 모든 키가 담긴 `Enumeration`을 `반환` |
| void save(OutputStream out, String header) | deprecated되었으므로 store()를 사용 |
| Object setProperty(String key, String value) | 지정된 `키`와 `값`(value)을 `저장`, 이미 존재하는 키면 `새로운 값`(value)으로 바꾼다. |
| void store(OutputStream out, String comments) | 저장된 `목록`을 지정된 OutputStream에 `출력`(저장) </br> comments는 목록에 대한 주석으로 저장 |
| void store(Writer writer, String comments) | 저장된 `목록`을 지정된 Writer에 `출력`(저장) </br> comments는 목록에 대한 주석(설명)으로 저장 |
| void storeToXML(OutputStream os, String comment) | 저장된 `목록`을 지정된 출력스트림에 XML문서로 `출력`(저장) </br> comments는 목록에 대한 설명(주석)으로 저장 |
| void storeToXML(OutputStream os, String comment, String encoding) | 저장된 `목록`을 지정된 출력스트림에 해당 인코딩의 XML문서로 `출력`(저장) </br> comments는 목록에 대한 설명(주석)으로 저장 |
| Set stringPropertyNames() | Properties에 저장되어 있는 `모든 키`를 `Set에 담아서 반환` |
> - setProperty()는 단순히 Hashtable의 put메서드를 호출할 뿐이며 기존에 같은 키로 저장된 값이 있는 경우 </br> 그 값을 `Object타입`으로 `반환`하며 그렇지 않으면 `null`을 `반환`한다. </br> 
> - Properties는 컬렉션 프레임웍 이전의 구버전이므로 Iterator가 아닌 Enumeration을 사용한다. (기능은 같다.)

## 5. Collections
- 컬렉션을 위한 메서드(static)를 제공
> `주의` : `java.util.Collection`은 `인터페이스`고 `java.util.Collections`는 `클래스`이다. 

### 5.1 컬렉션 채우기, 복사, 정렬, 검색 ...
- fill(), copy(), sort(), bonarySearch() ...

### 5.2 컬렉션의 동기화
- `synchronizedXXX()`   
: 기존의 Vector와 Hashtable과 같은 구버전의 클래스들은 동기화 처리가 되어있었지만 멀티쓰레드가 아닌 </br> 싱글쓰레드의 경우 동기화 기능으로 인해 성능을 떨어뜨리기때문에 
신버전의 클래스들은 동기화를 자체적으로 처리하지 않고 </br> `java.util.Collections`클래스의 동기화 메서드를 이용해서 동기화처리가 가능하도록 변경하였다.

```java
// 정의
static Collection synchronizedCollection(Collection c)
static List synchronizedList(List list)
static Set synchronizedSet(Set s)
static Map synchronizedMap(Map m)
static SortedSet synchronizedSortedSet(SortedSet s)
static SortedMap synchronizedSortedMap(Sortedmap m)
```
```java
// 사용방법
// 동기화 되지않은 ArrayList를 동기화햐여 synList에 저장
List sycList = Collections.synchronizedList(new ArrayList(...));
```

### 5.3 변경불가(readOnly) 컬렉션 만들기
- `unmodifiableXXX()`   
: 컬렉션을 보호하고자 할때 사용 (like 상수의 final과 같은 개념)

```java
// 정의
static Collection unmodifiableCollection (Collection c)
static List unmodifiableList (List list)
static Set unmodifiableSet (Set s)
static Map unmodifiableMap (Map m)
static NavigableSet unmodifiableNavigableSet (NavigableSet s)
static SortedSet unmodifiableSortedSet (SortedSet s)
static NavigableMap unmodifiable (NavigableMap m)
static SortedMap unmodifiable (SortedMap m)
```

### 5.4 싱글톤 컬렉션 만들기
- `singletonXXX()`   
: 객체 1개만 저장하는 컬렉션

```java
static List singletonList(Object o)
static Set singleton(Object o)
static Map singletomMap(Object key, Object value)
```

### 4.5 체크드 컬렉션 만들기
- `checkedXXX()`   
: 한 종류의 객체만 저장하는 컬렉션 만들기

```java
static Collection   checkedCollection(Collection c, Class type)
static List         checkedList(List list, Class type)
static Set          checkedSet(Set s, Class type)
static Map          checkedMap(Map m, Class keyType, Class valueType)
static Queue        checkedQueue(Queue queue, Class type)
static NavigableSet checkedNavigableSet(NavigableSet s, Class type)
static SortedSet    checkedSortedSet(SortedSet s, Class type)
static NavigableMap checkedNavigablemap(NavigableMap m, Class keyType, Class valueType)
static SortedMap    checkedSoredMap(SoredMap m, Class keyType, Class valueType)
```
> jdk 1.5부터 지네릭스가 나와 실제로 checked를 사용하는일은 거의 없다.

### 5.6 이 밖의 Collections 클래스의 메서드
```java
// 예제 

import java.util.*;
// Collections의 메서드를 사용할 때 Collections클래스 생략 가능
import static java.util.Collections.*;

public class CollectionsEx {

	public static void main(String[] args) {
		// list 생성
		List list = new ArrayList();
		System.out.println(list);
		// list에 값 저장
		addAll(list, 1,2,3,4,5);	// 가변인자이다.
		System.out.println(list);
		// 오른쪽으로 2칸 씩 이동
		rotate(list, 2);
		System.out.println(list);
		// 첫 번째와 세 번째 교환
		swap(list, 0, 2);
		System.out.println(list);
		// 무작위로 섞기
		shuffle(list);
		System.out.println(list);
		// 역순 정렬
		sort(list, reverseOrder());	// reverse(list)와 같은 결과
		System.out.println(list);
		// 정렬
		sort(list);
		System.out.println(list);
		// 이진탐색으로 3의 index값을 반환
		int idx = binarySearch(list, 3);	// binarySearch를 하기전에는 반드시 정렬을 한다.
		System.out.println("index of 3 = " + idx);
		// list에서 최대, 최소 반환
		System.out.println("max = " + max(list));
		System.out.println("min = " + min(list));
		System.out.println("min = " + max(list, reverseOrder()));
		// list를 9로 채운다.
		fill(list, 9);
		System.out.println("list = " + list);
		// list와 같은 크기의 새로운 list를 생성하고 2로 채운다. 결과는 변경불가
		List newList = nCopies(list.size(), 2);
		System.out.println("newList = " + newList);
		// list안에 newList와 공통된 요소가 있는지 확인, 공통 요소가 없으면 true
		System.out.println(disjoint(list, newList));
		// newList를 list에 복사
		copy(list, newList);
		System.out.println("newList = " + newList);
		System.out.println("list = " + list);
		// list의 내용을 2에서 1로 바꾼다.
		replaceAll(list, 2, 1);
		System.out.println("liist = " + list);
		// list에서 enumeration을 얻을 때 사용
		Enumeration e = enumeration(list);
		ArrayList list2 = list(e);
		
		System.out.println("list2 = " + list2);
	}

}
/*
[1, 2, 3, 4, 5]
[4, 5, 1, 2, 3]
[1, 5, 4, 2, 3]
[4, 5, 3, 1, 2]
[5, 4, 3, 2, 1]
[1, 2, 3, 4, 5]
index of 3 = 2
max = 5
min = 1
min = 1
list = [9, 9, 9, 9, 9]
newList = [2, 2, 2, 2, 2]
true
newList = [2, 2, 2, 2, 2]
list = [2, 2, 2, 2, 2]
liist = [1, 1, 1, 1, 1]
list2 = [1, 1, 1, 1, 1]
*/
```

# Reference
> 자바의 정석 - 남궁성

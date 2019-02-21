\#java #collection #map

---

## Map

### #구조?

- 키와 값으로 구성된 Entry객체를 저장하는 구조
- 키는 중복될 수 없으나, 값은 중복이 가능
- 키는 기본타입을 사용할 수 없음



### #종류?

- HashMap
  - Key를 해시코드구조로 구성
    - hashCode() 메소드를 호출해서 해시코드를 얻어내고 비교함
    - 만약 해시코드가 동일하면, equals() 메소드를 이용해 두객체를 다시 비교함
- Hashtable
  - HashMap과 동일한 내부구조를 가짐
  - 동가화된 메소드로 구성되서 스레드안전
- Properties
  - Hashtable의 하위클래스로 동일한 내부구조를 가짐
  - 키값은 String으로 제한
  - 애플리케이션 옵션정보, DB정보, 국제화정보가 저장된 .properties 파일을 읽을 때 사용
- TreeMap
  - 이진트리를 기반으로 검색기능을 강화시킨 Map 컬렉션



### #API(Map)

- 추가
  - put(K Key, V value) -> V 
- 검색
  - containsKey(Object key) -> boolean 
  - containsValue(Object value) -> boolean
  - get(Object key) -> V
  - isEmpty() -> boolean
  - entrySet() -> Set\<Map.Entry\<K,V>>
  - keySet() -> Set\<K> :: 중복불가 Set
  - values() -> Collection\<V> :: 중복가능 Collection
  - size() -> int
- 삭제
  - clear() -> void
  - remove(Object key) -> boolean



### #API(TreeMap)

- 검색
  - firstEntry() -> Map.Entry<K,V>
    - 제일 낮은 객체리턴
  - lastEntry() -> Map.Entry<K,V>
    - 제일 높은 객체리턴
  - lowerEntry(E e) -> Map.Entry<K,V>
    - 주어진 객체 바로 아래 객체리턴
  - higherEntry(E e) -> Map.Entry<K,V>
    - 주어진 객체 바로 위에 객체리턴
  - floorEntry(E e) -> Map.Entry<K,V>
    - 주어진 객체와 동등한 객체가 있으면 리턴, 없으면 바로 아래 객체 리턴
  - ceilingEntry(E e) -> Map.Entry<K,V>
    - 주어진 객체와 동등한 객체가 있으면 리턴, 없으면 바로 위 객체 리턴
  - pollFirstEntry() -> Map.Entry<K,V>
    - 제일 낮은 객체의 컬렉션 제거
  - pollLastEntry() -> Map.Entry<K,V>
    - 제일 높은 객체의 컬렉션 제거
- 정렬 
  - descendingKeySet() -> Iterator\<E>
    - 내림차순으로 정렬된 Key Set 리턴
  - descendingMap() -> NavigableMap\<K,V>
    - 내림차순으로 정렬된 Map 리턴
- 범위검색
  - headMap(K toKey, boolean inclusive) -> NavigableMap<K,V>
    - 주어진 객체보다 낮은 객체들을 리턴, 2번째 매개값으로 자신의 포함여부를 나타냄
  - tailMap(K fromKey, boolean inclusive) -> NavigableMap<K,V>
    - 주어진 객체보다 높은 객체들을 리턴, 2번째 매개값으로 자신의 포함여부를 나타냄
  - subMap(K fromKey, boolean fromInclusive, K toKey, boolean toInclusive) -> NavigableMap<K,V>
    - 주어진 두 객체 사이의 객체들을 리턴



## Code

### #HashMap

```java
Map<String, Integer> map = new HashMap<String, Integer>();
```

### #Hashtable

```java
Map<String, Integer> map = new Hashtable<String, Integer>();
```

### #Properties

> Properties 객체생성

```java
Properties properties = new Properties();
properties.load(new FileReader("C:/~/database.properties"));
```

> A 클래스 파일과 동등한 위치에 있는 database.properties 파일읽기

```java
String path = A.class.getResource("database.properties").getPath();
path = URLDecoder.decode(path, "utf-8"); // 한글 decode
Properties properties = new Properties();
properties.load(new FileReader(path));
```

> A 클래스 파일의 위치에서 경로를 수정할때

```java
String path = A.class.getResource("config/database.properties").getPath();
```

### #TreeMap


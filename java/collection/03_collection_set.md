\#java \#collection #set

---

## Set

### #구조?

- 주머니에 구슬을 담는 것과 같이 집합과 유사한 자료구조
- 저장순서가 유지되지 않으며(인덱스없음), 객체를 중복해서 저장할 수 없음
- null 또한 1개만 저장이 가능



### #종류?

- HashSet / LinkedHashSet
  - 해시코드구조로 구성
    - hashCode() 메소드를 호출해서 해시코드를 얻어내고 비교함
    - 만약 해시코드가 동일하면, equals() 메소드를 이용해 두객체를 다시 비교함
  - 필요에 따라 hashCode() & equals() 메소드를 오버라이드해서 사용 가능
- TreeSet
  - 검색기능을 강화시킨 Set 컬렉션으로 이진트리구조로 구성



### #API (Set)

- 추가
  - add(E element) -> boolean
- 검색
  - contains(Object o) -> boolean 
  - isEmpty() -> boolean
  - iterator() -> iterator\<E>
  - size() -> int 
- 삭제
  - clear() -> void
  - remove(Object o) -> boolean



### #API (TreeSet)

- 검색
  - first() -> E
    - 제일 낮은 객체를 리턴
  - last() -> E
    - 제일 높은 객체를 리턴
  - lower(E e) -> E
    - 주어진 객체보다 바로 아래 객체를 리턴
  - higher(E e) -> E
    - 주어진 객체보다 바로 위 객체를 리턴
  - floor(E e) -> E
    - 주어진 객체와 동등한 객체가 있으면 리턴, 없으면 바로 아래 리턴
  - ceiling(E e) -> E
    - 주어진 객체와 동등한 객체가 있으면 리턴, 없으면 바로 위 리턴
  - pollFirst() -> E
    - 제일 낮은 객체, 컬렉션 제거
  - pollLast() -> E
    - 제일 높은 객체, 컬랙션 제거
- 정렬
  - descendingIterator() -> Iterator\<E>
    - 내림차순으로 정렬된 Iterator 리턴
  - descendingSet() -> NavigableSet\<E>
    - 내림차순으로 정렬된 Set 리턴
- 범위검색
  - headSet(E toElement, boolean inclusive) -> NavigableSet\<E>
    - 주어진 객체보다 낮은 객체들을 리턴, 2번째 매개값으로 자신의 포함여부를 나타냄
  - tailSet(E fromElement, boolean inclusive) -> NavigableSet\<E>
    - 주어진 객체보다 높은 객체들을 리턴, 2번째 매개값으로 자신의 포함여부를 나타냄
  - subSet(E fromElement, boolean fromInclusive, E toElemet, boolean toInclusive) -> NavigableSet\<E>
    - 시작과 끝으로 주어진 객체들을 리턴



### #Iterator

- Iterator는 저장된 객체들을 한번씩 가져오는 역할

> Set -> Iterator

```java
Set<String> set = ...;
Iterator<String> iterator = set.iterator();
```

> API

- hasNext() -> boolean
- next() -> E
- remove() -> void

> Loop (Iterator)

```java
Set<String> set = ...;
Iterator<String> iterator = set.iterator();
while(iterator.hasNext()) {
    String str = iterator.next();
}
```

> Loop (Set)

```java
Set<String> set = ...;
for(String str : set) {
    
}
```



## Code

### #HashSet

> 생성

```java
Set<String> set = new HashSet<String>();
```

> hashCode() & equals() 오버라이드

```java
public class Member {
    public String name;
    public int age;
    
    public Member(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    @Override
    public boolean equals(Object o) {
        if(o instanceof Member) {
            Member member = (Member) o;
            return member.name.equals(name) && (member.age == age);
        } else {
            return false;
        }
    }
    
    @Override
    public int hashCode() {
        return name.hashCode() + age;
    }
}
```

### #TreeSet

```java
TreeSet<E> treeSet = new TreeSet<E>();
```


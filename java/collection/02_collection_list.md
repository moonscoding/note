\#java #collection #list

---

## List

### #구조?

- 순서가 있는 선형구조의 자료구조



### #종류?

- ArrayList
  - Array와 동일한 자료구조로 인덱스로 관리 
  - 용량이 초과되면 자동으로 용량을 늘림
    - 검색에 용이 / 순차적인 추가&삭제에 용이
- Vector
  - ArrayList와 동일한 구조를 가지며, 스레드안전을 보장
- LinkedList
  - 다음객체의 참조를 통해서 체인처럼 관리
    - 규칙없는 추가&삭제에 용이



### #API

- 추가

  - add(E element) -> boolean
  - add(int index, E element) -> void
  - set(int index, E element) -> void

- 검색

  - contains(Object o) -> boolean
  - get(int index) -> E 
  - isEmpty() -> boolean
  - size() -> int

- 삭제

  - clear() -> void
  - remove(int index) -> E
  - remove(Object o) -> boolean

  

### #Arrays.asList()

- add/remove 불가
  - UnsupportedOperationException



## Code

### #ArrayList

- 10개의 기본 저장용량을 가짐

```java
List<String> list = new ArrayList<String>(); // 10
List<String> list = new ArrayList<String>(30);
List list = new ArrayList(); // 성능상 좋지 않음
```



### #Vector

- 동기화된 메소드로 구성

```java
List<String> list = new Vector<String>();
```



### #LinkedList

```java
List<E> list = new LinkedList<E>();
```


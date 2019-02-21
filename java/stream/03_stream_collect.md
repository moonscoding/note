\#java \#stream \#collect

---



### 수집처리

###### \#collect()

- collect(Collector<T,A,R> collector)
- T 요소를 A 누적기가 R에 수집하는 원리




- [EX] 데이터

```java
List<Student> totalList = Arrays.asList(
    new Student("nameA", 10, Student.Zender.MALE, Student.City.Seoul),
    new Student("nameB", 20, Student.Zender.FEMALE, Student.City.Seoul),
    new Student("nameC", 30, Student.Zender.MALE, Student.City.Pusan),
    new Student("nameD", 40, Student.Zender.FEMALE, Student.City.Pusan),
    new Student("nameE", 50, Student.Zender.MALE, Student.City.Pusan)
);
```



### List 수집

###### #Collectors.toList()

```java
List<Student> maleList = totalList.stream()
    .filter(s -> s.getZender() == Student.Zender.MALE)
    .collect(Collectors.toList());
```



### Set 수집

###### \#Collectors.toSet()

```java
List<Student> maleSet = totalList.stream()
    .filter(s -> s.getZender() == Student.Zender.MALE)
    .collect(Collectors.toSet());
```



### Map 수집

###### \#Collectors.toMap()

- toMap(Function\<T, K\> keyMapper, Function\<T, U\> valueMapper)

```java
List<Student> femaleMap = totalList.stream()
    .filter(s -> s.getZender() == Student.Zender.FAMELE)
    // s -> s == Function.identity()
    .collect(Collectors.toMap(s -> s.getName(), s -> s));
```

###### \#Collectors.toConcurrentMap()

- toMap(Function\<T, K\> keyMapper, Function\<T, U\> valueMapper)

```java
...
```



### Collection 수집

###### #Collectors.toCollection()

- toCollection(Supplier\<Collection\<T\>\>)
  - T를 Supplier가 제공한 Collection에 저장

- [EX] toCollection(HashSet::new)

```java
List<Student> femaleSet = totalList.stream()
    .filter(s -> s.getZender() == Student.Zender.FAMALE)
    .collect(Collectors.toCollection(HashSet::new));
```



### Custon 수집

###### #custom

```java
MaleStudent maleStudent = totalList.stream()
    .filter(s -> s.getZender() == Student.Zender.MALE)
    .collect(MaleStudent::new, MaleStudent::accumulate, MaleStudent::combine);
```

```java
class MaleStudent {
    private List<Student> list; // 컬렉션

    public MaleStudent() {
        list = new ArrayList<Student>();
        System.out.println("[" + Thread.currentThread().getName() + "] MaleStudent()");
    }

    // 요소수집 메소드
    public void accumulate(Student student) {
        list.add(student);
        System.out.println("[" + Thread.currentThread().getName() + "] accumulate()");
    }

    // 결합 메소드 ( 병렬처리시에만 호출 )
    public void combine(MaleStudent other) {
        list.addAll(other.getList());
        System.out.println("[" + Thread.currentThread().getName() + "] combine()");
    }

    public List<Student> getList() {
        return list;
    }

    public void setList(List<Student> list) {
        this.list = list;
    }
}
```


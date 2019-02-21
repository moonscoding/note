\#java \#stream \#filter

---

### 파이브라인 (남성평균나이)

###### \#filter() 

###### \#mapToInt() 

###### \#average() 

###### \#getAsDouble()

```java
List<Student> list = Arrays.asList(
    new Student("nameA", Student.MALE, 30),
    new Student("nameB", Student.FEMALE, 40),
    new Student("nameC", Student.MALE, 50),
    new Student("nameD", Student.FEMALE, 60),
    new Student("nameE", Student.MALE, 70)
);

double avgOfAge = list.stream()
    .filter(m -> m.getZender() == Student.MALE)
    .mapToInt(Student::getAge)
    .average()
    .getAsDouble();
```



### 필터링

###### \#distinct() 

###### \#filter()

```java
List<String> list = Arrays.asList("A", "A", "B", "B", "C");
list.stream()
    .distinct()
    .forEach(n -> System.out.println(n));

list.stream()
    .filter(n -> n.starsWith("C"))
    .forEach(n -> System.out.println(n));

list.stream()
    .distinct()
	.filter(n -> n.starsWith("C"))
    .forEach(n -> System.out.println(n));
```



### 매핑처리 

###### #map() #mapToInt() #mapToLong() #mapToDouble() #mapToObj()

```java
List<Student> list = Arrays.asList(
    new Student("nameA", Student.MALE, 30),
    new Student("nameB", Student.FEMALE, 40),
    new Student("nameC", Student.MALE, 50),
    new Student("nameD", Student.FEMALE, 60),
    new Student("nameE", Student.MALE, 70)
);

list.stream()
    .mapToInt(Student::getAge)
    .forEach(age -> System.out.println(age));
```

###### \#flatMap() 

```java
List<String> inputList = Arrays.asList("java8 lambda", "stream mapping");
inputListA.stream()
    // string -> stream
    .flatMap(data -> Arrays.stream(data.split(" ")))
    .forEach(word -> System.out.println(word));
```

###### \#flatMapToInt() 

```java
List<String> inputList = Arrays.asList("10, 20, 30", "40, 50, 60"); 
inputList.stream()
    // string -> stream
    .flatMapToInt(data -> {
        String[] strArr = data.split(",");
        int[] intArr = new int[strArr.length];
        for(int i=0; i<strArr.length; i++) {
            intArr[i] = Integer.parseInt(strArr[i].trim());
        }
        return Arrays.stream(intArr);
    })
    .forEach(n -> System.out.println(n));
```



### 변환처리

###### \#asDoubleStream()

- int -> double
- long -> double

```java
int[] intArray = {1,2,3,4,5};
IntStream intStream = Arrays.stream(intArray);
intStream
	// IntStream -> DoubleStream
	.asDoubleStream()
    .forEach(d -> System.out.println(d));
```

###### #asLongStream()

- int -> long

```java
...
```

######  \#boxed()

 - int -> Integer
 - long -> Long
 - double -> Double

```java
int[] intArray = {1,2,3,4,5};
IntStream intStream = Arrays.stream(intArray);
intStream
	.boxed()
    .forEach(obj -> System.out.println(obj.intValue()));
```



### 집계처리

###### \#count() 

- :: long

```java
...
```

###### \#findFirst() 

- :: OptionalXXX

```java
...
```

###### \#max(Comparator\<T\>) 

- :: Optional\<T\>

```java
...
```

###### \#max() 

- :: OptionalXXX

```java
...
```

###### \#min(Comparator\<T\>) 

- :: Optional\<T\>

```java
...
```

###### \#min()

- :: OptionalXXX

```java
...
```

###### \#average() 

- :: OptionalDouble

```java
...
```

###### \#sum() 

- :: int, long, double

```java
...
```



### 정렬처리

###### \#sorted()

- 숫자배열정렬

```java
IntStream intStream = Arrays.stream(new int[] {5,4,3,2,1});
intStream
	.sorted()
    .forEach(n -> System.out.println(n));
```

- 객체배열정렬
  - `Comparable` 인터페이스를 상속하고 `compareTo()` 메소드를 구현한 클래스만 정렬가능

```java
List<Student> students = Arrays.asList(
    new Student("nameA", Student.MALE, 50),
    new Student("nameB", Student.FEMALE, 40),
    new Student("nameC", Student.MALE, 30),
    new Student("nameD", Student.FEMALE, 20),
    new Student("nameE", Student.MALE, 10)
);

students.stream()
    .sorted()
    .forEach(s -> System.out.println(s.getAge()));
```



### 루핑처리 - 중간단계확인

###### \#peek()

```java
int[] arr = new int[] {1,2,3,4,5};

Arrays.stream(arr)
    .filter(a -> a%2 == 0)
    .peek(n -> System.out.println(n)); // 동작 X 

int sum = Arrays.stream(arr)
    .filter(a -> a%2 == 0)
    .peek(n -> System.out.println(n)) // 동작 O
   	.sum(); 
```



### 일치확인

###### \#allMatch() 

```java
int[] arr = {2,4,6};
boolean result = Arrays.stream(arr)
    .allMatch(a -> a%2 == 0); // true
```

###### \#anyMatch() 

```java
int[] arr = {2,4,6};
boolean result = Arrays.stream(arr)
    .allMatch(a -> a%3 == 0); // true
```

###### \#noneMatch()

```java
int[] arr = {2,4,6};
boolean result = Arrays.stream(arr)
    .allMatch(a -> a%3 == 0); // false
```



### 예외처리

###### \#orElse()

```java
List<Integer> list = new ArrayList<>();
double avg = list.stream()
    .mapToInt(Integer::intValue)
    .average()
    .orElse(0.0);
```

###### \#ifPresent()

```java
List<Integer> list = new ArrayList<>();
list.stream()
    .mapToInt(Integer->intValue)
    .average()
    .ifPresent(a -> System.out.println(a));
```



### 커스텀집계

###### \#reduce()

```java
List<Student> list = Arrays.asList(
    new Student("nameA", Student.MALE, 30),
    new Student("nameB", Student.FEMALE, 40),
    new Student("nameC", Student.MALE, 50)
);

int sumA = list.stream()
    .mapToInt(Student::getAge)
    .sum();

int sumB = list.stream()
    .map(Student::getAge)
    .reduce((a,b) -> a+b)
    .get();

int sumC = list.stream()
    .map(Student::getAge)
    .reduce(0, (a,b) -> a+b);
```


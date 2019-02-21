#java \#stream \#group

---



### 그룹처리

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



###### #Collectors.groupingBy()

 - :: Map

```java
Map<Student.Zender, List<Student>> mapByZender = totalList.stream()
    .collect(Collectors.groupingBy(Student::getZender));
```



### 집계처리 (그룹처리후)

###### #Collectors.counting()
```java
...
```



###### #Collectors.averagingDouble()

```java
Map<Student, Double> mapByZender = totalList.stream()
    .collect(Collectors.groupingBy(
    	Student::getZender,
        Collectors.averagingDouble(Student::getAge)
    ));
```



###### #Collectors.maxBy() #Collectors.minBy()

```java
...
```



###### #summingInt() #summingLong() #summingDouble()

```java
...
```



### 매핑처리 (그룹처리후)

###### #Collectors.mapping()

```java
Map<Student.Citry, List<String>> mapByCity = totalList.stream()
    .collect(Collectors.groupingBy(
    	Student::getCity,
        Collectors.mapping(Student::getName, Collectors.toList())
    ));
```



###### #Collectors.joining()

- :: String

```java
Map<Student.Zender, String> mapByZender = totalList.stream()
    .collect(Collectors.groupingBy(
    	Student::getZender,
        Collectors.mapping(
        	Student::getName,
            Collectors.joining(",")
        )
    ));
```










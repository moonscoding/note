# Reflection



> Spring의 Dependency Injection은 어떻게 동작할까?
>
> - 이번 장에서는 Reflection의 원리 및 기능에 대해서 정리

```java
@Service
public class BookService {
    
    @Autowired 
    BookRepository bookRepository;
}
```



## Class

- 모든 클래스를 로딩한 다음 Class<T>의 인스턴스가 생기며, `${ClassName}.class`로  접근할 수 있음
  - 클래스 인스턴스이기 때문에 힙에 저장

- 모든 인스턴스는 `getClass()` 메소드를 가짐

- `Class.forName("${FQCN}")`을 통해서 클래스를 문자열로 읽을 수 있음
  - 클래스 패스에 해당 클래스가 없다면 `ClassNotFoundException` 발생



> Class<T>가 할 수 있는 것

- 필드(목록) 가져오기
- 메소드(목록) 가져오기
- 애노테이션(목록) 가져오기
- 생성자 가져오기
- 상위 클래스 가져오기
- 인터페이스(목록) 가져오기
- ... 



## Method



## Field



## Annotation



> Annotation 기본정리 (별도 정리후 링크 연결 예정)

Annotation의 기본은 주석이기 때문에 런타임시에 Reflection을 활용하여 읽을 수 없음

따라서 런타임에 활용할 수 있도록 Retention 속성을 변경 필요

```java
@Retention(RetentionPolicy.CLASS)
@Retention(RetentionPolicy.RUNTIME)
```

Annotation 범위 설정

```java
@Target(ElementType.TYPE, ElementType.FIELD)
```

Annotation 상속 여부

```java
@Inherit
```



### getAnnotations()

상속받은(@Inherit) 애노테이션까지 조회



### getDeclaredAnnotations()

자기 자신에만 붙어있는 애노테이션 조회





###### 참조

https://www.inflearn.com/course/the-java-code-manipulation
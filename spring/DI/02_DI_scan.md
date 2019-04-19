\#spring #DI #scan

---



## 컴포넌트스캔

- 컴포넌트의 스캔 범위를 지정 (성능향상)

### #탐색범위

- 탐색범위는 자세한 것이 유리
- basePackages = value

```java
@ComponentScan(basePackages="com.example.demo")
@ComponentScan(value="com.example.demo.app")
```



> 자바기반

```java
...
```

> Annotation기반

```java
@Configuration
@Component("com.example.demo")
public class AppConfig {
    
}
```

> XML기반

```xml
<beans>
  <context:component-scan base-package="com.example.demo" />
</beans>
```





## 기본컴포넌트

- 컴포넌트 스캔은 클래더 로더를 스캔하며 특정 클래스를 찾은 후, DI 컨테이너에 등록하는 과정까지를 말함



### #종류?

- @Component
  - 유틸리티 컴포넌트
- @Controller 
  - MVC 패턴에서 C, 컨트롤러역할
  - 클라이언트의 요청을 받고 비즈니스 로직(@Service)의 처리 결과를 응답으로 돌려보냄
- @RestController 
  - REST API를 처리하기 위한 컨트롤러
- @Service 
  - 비즈니스 로직을 담당
  - 컨트롤러에게 데이터를 전달받아 실행
  - 영속적인 데이터는 @Repository로 위임
- @Repository 
  - 영속적인 데이터 처리를 수행
  - ORM관련 라이브러리로 데이터의 CRUD 처리
- @Configuration 
  - 환경구성을 수행
- @ControllerAdvice 
  - 컨트롤러의 예외처리를 담당
- @ManageBean 
  - ...
- @Named
  - ...



## 필터

- 기본컴포넌트 외에 다른 컴포넌트를 포함하고 싶다면 필터를 적용해 스캔범위를 커스터마이징 가능



### #종류?

- 애너테이션 활용 필터 (ANNOTATION)
- 할당 가능한 타입을 활용한 필터 (ASSIGNABLE_TYPE)
- 정규 표현식 패턴을 활용한 필터 (REGEX)
- AspectJ 패턴을 활용한 필터 (ASPECTJ)



### #ASSIGNABLE_TYPE

> 인터페이스기반

```java
public interface DemainService{}
```

> 자바기반

```java
@ComponentScan(
    basePackages="com.example.demo",
    includeFilters={@ComponentScan.Filter(type=FilterType.ASSIGNABLE_TYPE, classes={DomainService.class})}
)
```

> XML기반

```xml
<context:component-scan base-package="com.example.demo">
  <context:include-filter type="assignable" expression="com.example.demo.domain.DomainService" />
</context:component-scan>
```



### #REGEX

> 자바기반

```java
@ComponentScan(
  basePackages="com.example.demo",
  includeFilters={@ComponentScan.Filter(type=FilterType.REGEX, pattern={".+DomainService$"})}
)
```

> XML기반

```xml
<context:component-scan base-package="com.example.demo">
  <context:include-filter type="regex" expression=".+DomainService$" />
</context:component-scan>
```



### #userDefaultFilters

- 필터적용시, 애너테이션이 붙은 스캔대상도 탐색범위에 포함
- 애너테이션이 붙은 스캔대상을 무시하고 필터적용대상만 조회하고 싶을 때
- default : true

> 자바기반

```java
@ComponentScan(
  basePackages="com.example.demo",
  userDefaultFilters=false,
  includeFilters={@ComponentScan.Filter(type=FilterType.REGEX, pattern={".+DomainService$"})}
)
```

> XML기반

```xml
<context:component-scan base-package="com.example.demo" use-default-filters="false">
  <context:include-filter type="regex" expression=".+DomainService$" />
</context:component-scan>
```



### #excludeFilters

> 자바기반

```java
@ComponentScan(
  basePackages="com.example.demo",
  userDefaultFilters=false,
  includeFilters={@ComponentScan.Filter( type=FilterType.REGEX, pattern={".+DomainService$"})},
  excludeFilters={@ComponentScan.Filter( type=FilterType.ANNOTATION, pattern={Exclude.class})}
)
```

> XML기반

```xml
context:component-scan base-package="com.example.demo" use-default-filters="false">
  <context:include-filter type="regex" expression=".+DomainService$" />
  <context:exclude-filter type="annotation" expression="com.example.demo.Exclude" />
</context:component-scan>
```


\#spring #databinding

## 데이터바인딩

### #용어?

- 데이터바인딩
  - 자바객체의 프로퍼티에서 외부에서 입력된 값을 설정하는 과정
- 자바빈즈
  - 데이터바인딩이나 프로퍼티관점에서 다뤄지는 객체

> 데이터바인딩을 사용하지 않았을때

```java
public class EmployeeForm {
  private String name;
  private Integer joinedYear;
  // ...
}
```

```java
EmployeeForm form = new EmployeeForm();
form.setName(request.getParameter("name"));
form.setJoinedYear(Integer.valueOf(request.getParameter("joinedYear"))); // 형변환
```

- `문제` 만약 프로퍼티의 개수가 늘어나면 그에 비례해서 데이터바인딩 코드도 늘어남



### #String 데이터바인딩

- `ServletRequestDataBinder` 클래스 사용
- 매개변수의 개수 제약이 없음
  - `MVC 데이터바인딩`을 활용하면 더 간단히 처리가 가능

```java
EmployeeForm form = new EmployeeForm();
ServletRequestDataBinder dataBinder = new ServletRequestDataBinder(form);
dataBinder.bind(request);
```



### #Spring 형변환

- 데이터바인딩을 처리시 자바빈즈 프로퍼티 타입에 맞게 입력된 문자열 값을 형변환 (String -> 알맞은타입)

#### #빈조작방식

- 빈을 조작하고 래핑하는 방식
  - `BeanWrapper` 인터페이스는 개발자가 직접 다룰 일이 적고, 
    스프링 내부에서 DataBinder 클래스나 BeanFactory 인터페이스가 활용
  - 대신, `PropertyEditor` 인터페이스의 구현클래스를 활용해 형변환 
    (변환 전 값이 String으로 한정)

##### #PropertyEditor

```bash
application.healthCheck=yes
```

```java
@Component
public class ApplicationProperties {
  @Value("${application.healthCheck:no}")
  private boolean healthCheckEnabled; // true 설정
}
```

#### #형변환

- `Converter `인터페이스등의 구현클래스를 활용해 형변환
  - `PropertyEditor`는 변환전 값이 `String`으로 제한
  - `Converter`는 String 외 타입도 가능

##### #ConversionService

- `형변환` & `필드값표현형식 포매팅` 기능은 다은 인터페이스로 제공
  - 다양한 구현클래스중  `DefaultFormattingConvertionService` 사용이 일반적

###### #정의

```java
@Bean
public ConversionService conversionService() {
  return new DefaultFormattingConvertionService();
}
```

###### #예제

```bash
application.dataOfServiceStarting=2017-05-10
```

```java
@Component
public class ApplicationProperties {
  @Value("${application.dataOfServiceStarting:}")
  private java.time.LocalDate dataOfServiceStarting; // 2017년 5월 10일로 설정
}
```

##### #Custom 형변환

- 직접 형변환기능을 만들어 사용
- `Converter` 인터페이스 직접 구현

> EmailValue 자바빈즈

```java
public class EmailValue {
  @Size(max = 256)
  @Email
  private String value;

  public void setValue(String value) {
    this.value = value;
  }

  public String getValue() {
    return value;
  }

  public String toString() {
    return getValue();
  }
}
```

> Converter 구현

```java
public class StringToEmailValueConverter implements Converter<String, EmailValue> {
  @Override
  public EmailValue convert(String source) {
    EmailValue email = new EmailValue();
    email.setValue(source);
    return email;
  }
}
```

> Custom Converter 추가

- addConverter

```java
@Bean
public ConversionService conversionService() {
  DefaultFormattingConvertionService conversionService =
    new DefaultFormattingConvertionService();
  // addConverter 메서드의 매개변수로 작성한 Converter를 지정
  conversionService.addConverter(new StringToEmailValueConverter());
  return conversionService;
}
```

> 사용

```bash
application.dateOfServiceStarting=20180808
```

```java
@Component
public class ApplicationProperties {

  @Value("$application.dateOfServiceStarting:")
  private java.time.LocalDate dateOfServiceStarting; // 2018 08 08 로 설정
}
```



#### #필드값표현방식 포매팅

- `Formatter` 인터페이스등 구현클래스를 활용
  - String과 임의의 클래스를 상호 변환하기 위한 인터페이스
  - 형변환시 Local을 고려한 국제화 기능 제공
  - 숫자와 날짜등 로컬에 따라 다른 포맷형태를 가진 클래스와 형변환에 사용

##### #포매팅 애너테이션

- `DefaultFormattingConvertionService`  사용할 때, 형변환 시 적용할 포맷형식을 정할 수 있음
  - 특정부분만 원하는 패턴을 명시해야 할 때

###### #@DataTimeFormat

###### #@NumberFormat

```java
@Component
public class ApplicationProperties {
  @Value("$application.dateOfServiceStarting:")
  @DataTimeFormat(pattern="yyyy/MM/dd") // 포맷명시
  private java.time.LocalDate dateOfServiceStarting;
}
```

##### #Custom 포매팅

- setDateFormatter()
- registerFormatter()

```java
@Bean
public ConversionService conversionService() {
  DefaultFormattingConvertionService conversionService =
    new DefaultFormattingConvertionService();
  DateTimeFormatterRegistrar registrar = new DateTimeFormatterRegistrar();
  // BASIC_ISO_DATE 형식사용
  registrar.setDateFormatter(DateTimeFormatter.BASIC_ISO_DATE);
  registrar.registerFormatter(conversionService);
  return conversionService;
}
```


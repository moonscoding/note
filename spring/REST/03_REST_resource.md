## #Resource

- 리소스 클래스는 JSON 이나 XML 형식의 데이터를 자바빈즈로 표현한 클래스
- 스프링 MVC는 리소스 클래스를 통해서 서버오 클라이언트 사이의 리소스 상태를 연계하는 역할을 합니다.

![1555661690499](assets/1555661690499.png) 

> 요청된 JSON

```java
{
  "bookId" : "bookId",
  "name" : "moon",
  "publishedDate" : "2018-08-20",
  "authors" : [ "tom", "bob", "jany" ],
  "publisher" : {
    "name" : "MoonsCoding",
    "tel" : "02-1234-5678"
  }
}
```

> 변환된 Resource
>
> - 주의할 점은 json 프로퍼티명과 resource 프로퍼티명이 같아야 한다는 점

```java
public class BookResource implements Serializable {
    private static final long serialVersionUID = -9115030674240690591L;

    private String bookId;
    private String name;
    private List<String> authors;
    @DateTimeFormat(pattern="yyyy-MM-dd")
    private LocalDate publishedDate;
    private BookPublisher publisher;

    public String getBookId() {
        return bookId;
    }

    public void setBookId(String bookId) {
        this.bookId = bookId;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public LocalDate getPublishedDate() {
        return publishedDate;
    }

    public void setPublishedDate(LocalDate publishedDate) {
        this.publishedDate = publishedDate;
    }

    public List<String> getAuthors() {
        return authors;
    }

    public void setAuthors(List<String> authors) {
        this.authors = authors;
    }

    public BookPublisher getPublisher() {
        return publisher;
    }

    public void setPublisher(BookPublisher publisher) {
        this.publisher = publisher;
    }

    public static class BookPublisher implements Serializable {

        private static final long serialVersionUID = -8119817744873562082L;

        private String name;
        private String tel;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public String getTel() {
            return tel;
        }

        public void setTel(String tel) {
            this.tel = tel;
        }
    }
}
```



## #Jackson

- 포맷제어에 이용한는 라이브러리
  - JSON 들여쓰기 설정
  - 언더스코어('\_')로 구분되는 JSON 필드를 다루는 법
  - Java SE8에서 추가된 Date and Time API 클래스를 지원
  - 날짜/시간 타입의 포맷을 지정



> 애너테이션 종류

- @JsonProperty
- @JsonIgnore
- @JsonInclude
- @JsonIgnoreProperties
- @JsonPropertyOrder
- @JsonSerialize
- @JsonDeserialize



> 기능

- ObjectMapper 
  - JSON <-> POJO 객체 변환

- Jackson2ObjectMapperBuilder
  - 빌드패턴을 이용해서 ObjectMapper를 만드는 클래스로 자바기반 설저방식으로 빈을 정의
- Jackson2ObjectMapperFactoryBean
  - 스프링이 제공하는 FactoryBean을 사용해서 ObjectMapper를 만드는 클래스로 주로 XML 기반 방식에서 빈을 정의시에 사용



### #Jackson2ObjectMapperBuilder

```Java
@Bean
ObjectMapper objectMapper() {
  return Jackson2ObjectMapperBuilder.json()
    // 옵션설정
    .build();
}
```

> JSON 들여쓰기 처리방법

```java
@Bean
ObjectMapper objectMapper() {
  return Jackson2ObjectMapperBuilder.json()
    .indentOutput(true)
    .build();
}
```

> Date and Time 포맷지정방법
>
> - `StdDateFormat` - ISO8601의 날짜/시간을 지원
>   - `LocalDate` yyyy-MM-dd
>   - `LocalTime` HH:mm:ss.SSS
>   - `LocalDateTime` yyyy-MM-dd'T'HH:mm:ss:SSS
>   - `ZonedDateTime` yyyy-MM-dd'T'HH:mm:ss:SSS'Z'

```java
@Bean
ObjectMapper objectMapper() {
  return Jackson2ObjectMapperBuilder.json()
    .indentOutput(true)
    .dateFormat(new StdDateFormat())
    .build();
}
```



### #Jackson 활용

> Json -> POJO

```java
// Json File에서 읽기
MyValue valueA = mapper.readValue(new File("data.json"), MyValue.class);

//  URL 에서 읽기
MyValue valueaB = mapper.readValue(new URL("http://some.com/api/entry.json"), MyValue.class);

// String 으로 읽기
MyValue valueC = mapper.readValue("{\"name\":\"Bob\", \"age\":13}", MyValue.class);
```

> POJO -> Json

```java
ObjectMapper mapper = new ObjectMapper();

MyValue  myResultObject = new MyValue();
myResultObject.name = "MyName";
myResultObject.age= 11;

// result.json 파일로 저장
mapper.writeValue(new File("result.json"), myResultObject);

// byte[] 로 저장
byte[] jsonBytes = mapper.writeValueAsBytes(myResultObject);

// string 으로 저장
String jsonString = mapper.writeValueAsString(myResultObject);
```
# Spring Boot Test

> 종류

|   Annotation    |             Desc             |     Bean      |
| :-------------: | :--------------------------: | :-----------: |
| @SpringBootTest |      통합 테스트, 전체       |   Bean 전체   |
|   @WebMvcTest   |    단위테스트, MVC 테스트    | MVC 관련 Bean |
|  @DataJpaTest   |    단위테스트, JPA 테스트    | JPA 관련 Bean |
| @RestClientTest | 단위 테스트, REST API 테스트 |   일부 Bean   |
|    @JsonTest    |   단위 테스트, Json 테스트   |   일부 Bean   |



## 통합 테스트

### @SpringBootTest

- 테스트에서 사용할 ApplicatioContext를 쉽게 생성하고 조작
- 기존 SpringTest의 @ContextConfiguration의 발전된 기능
- 전체 빈 중 특정 빈을 선택하여 생성
- 특정 빈을 Mock으로 대체
- 데스트에 사용할 프로퍼티 파일을 선택하거나 특정 속성만 추가
- 특정 Configuration을 선택하여 설정
- 주요 기능으로 테스트 웹 환경을 자동으로 설정해주는 기능
- @RunWith(SpringRunner.class)와 함께 사용



> <u>[통합테스트] 여러 단위 테스트를 하나의 통합된 테스트로 수행할 때 적합</u>

- 애플리케이션의 설정을 모두 로드 -> 애플리케이션 규모가 클수록 느려짐 (단위테스트라는 의미 희석)



#### classes 옵션

> Bean 정의 ( classes 옵션 )

```java
@RunWith(SpringRunner.class)
@SpringBootTest(
	classes = {ArticleServiceImpl.class, CommonConfig.class}	
)
public class SomeClassTest {
    @Autowired
    ArticleServiceImpl articleServiceImpl;
}
```

#### properties 옵션

설정이 기존과 달라질 필요가 있는 경우 이때를 위해 properties 속성을 바꾸는 옵션 제공

```java
@RunWith(SpringBoot.class)
@SpringBootTest(properties = "classpath:application-test.yml")
public class SomeTest {
    ...
}
```

#### webEnvironment

- `MOCK`
  - WebApplicationContext를 로드, 내장된 서블릿 컨테이너가 아닌 Mock 서블릿을 제공
  - @AutoConfigureMockMvc 어노테이션을 함께 사용하면 별다른 설정 없이 간편하게 MockMvc를 사용한 테스트를 진행
- `RANDOM_PORT`
  - EmbeddedWebApplicationContext를 로드 , 실제 서블릿 환경 구성
  - 생성된 서블릿 컨테이너는 임의의 포트 listen
- `DEFINED_PORT`
  - RAMDOM_PORT와 동일하게 실제 서블릿 환경을 구성
  - 포트는 애플리케이션 프로퍼티에서 지정한 포트를 listen
- `NONE`
  - 일반적 ApplicationContext를 로드 아무런 서블릿 환경을 구성하지 않음



### @TestConfiguration

```java

@TestConfiguration
public static class TestConfig {
    @Bean
    public RestTemplate restTemplate() {
        @Override
        public <T> getForObject(String url, Class<T> responseType, Object... uriVariables) 
            throws RestClientException { 
        	if(responseType == String.class) {
            	return (T) "Good";
            } else {
                throw new IllegalArgumentException();
            }
        }
    }
}
```

- ComponentScan을 통해 감지되어 만일 @SpringBootTest의 classes 속성을 이용하여 
  특정 클래스만을 지정했을 경우 TestConfiguration은 감지 되지 않음
- 그런 경우 classes 속성에 직접 TestConfiguration을 추가



### @Import

- 더 좋은 방법은 @Import 어노테이션을 사용하는 것
- @Import 어노테이션을 통해 직접 사용할 TestConfiguration을 명시할 수 있으며 
  특정 테스트 클래스의 내부 클래스가 아닌 별도의 클래스로 분리하여 여러 테스트 에서 공유 가능

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = ArticleServiceImpl.class)
@Import(TestConfig.class)
public class TestConfigArticleServiceImplTest {
    
    @MockBean
    private ArticleDao articleDao;
    
    @Autowired
    private RestTemplate restTemplate;
    
    @Autowired
    private ArticleServiceImpl articleServiceImpl;

    @Test
    public void test() {
        String good = restTemplate.getForObject("test", String.class);
        assertThat(good).isEqualTo("Good");
    }
}
```

## Mock Test

### @MockBean

- @MockBean 어노테이션을 사용해 이름 그대로 Mock 객체를 빈으로 등록 가능 
- 그렇기 때문에 만일 @MockBean으로 선언된 빈을 주입받으면(@Autowired같은 어노테이션 등을 통해서) 
  Spring ApplicationContet Mock 객체를 주입
- 새롭게 @MockBean을 선언하면 Mock 객체를 빈으로 등록, 
  만일 @MockBean으로 선언한 객체와 같은 이름과 타입으로 이미 빈으로 등록되어 있다면 해당 빈은 선언한 Mock 빈으로 대체

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = ArticleServiceImpl.class)
public class ArticleServiceImplTest {
    
    @MockBean
    private RestTemplate restTemplate;
    
    @MockBean
    private ArticleDao articleDao;
    
    @Autowired
    private ArticleServiceImpl articleServiceImpl;

    @Test
    public void testFindFromDB() {
        List<Article> expected = Arrays.asList(
                new Article(0, "author1", "title1", "content1", Timestamp.valueOf(LocalDateTime.now())),
                new Article(1, "author2", "title2", "content2", Timestamp.valueOf(LocalDateTime.now())));

        given(articleDao.findAll()).willReturn(expected);

        List<Article> articles = articleServiceImpl.findFromDB();
        assertThat(articles).isEqualTo(expected);
    }
}
```



> 일반화하여 사용가능한가 ?



### @SpyBean

- @SpyBean은 말그대로 스파이입니다.
- @MockBean은 given에서 선언한 코드 외에 전부 사용할 수 없습니다.
- 반면 @SpyBean은 given에서 선언한 코드 외에는 전부 실제 객체의 것을 사용 



## Client Test

### TestRestTemplate

> ClientSide API TEST

- RestTemplate의 테스트를 위한 버전
- @SpringBootTest에서 Web Environment 설정을 하였다면 TestRestTemplate은 그에 맞춰서 자동으로 설정
- 기존의 MockMvc와 어떤 차이 ?
  - Servlet Container를 사용하냐 하지 않느냐 차이
    - MockMvc는 Servlet Container를 생성하지 않습니다.
    - 반면 @SpringBootTest & TestRestTemplate은 Servlet Container를 사용 (마치 실제 서버 동작과 같이 테스트 수행 가능)
  - 테스트 관점의 차이
    - MockMvc는 서버 입장에서 구현된 API를 통해 비즈니스 로직이 문제 없이 수행되는지 테스트를 할 수 있다면,
    - TestRestTemplate은 클라이언트 입장에서 RestTempalte을 사용하듯이 테스트를 수행

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class RestApiTest {
    
    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    public void test() {
        ResponseEntity<Article> response = restTemplate.getForEntity("/api/articles/1", Article.class);
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response.getBody()).isNotNull();
        ...
    }
}
```



## Controller Test

### @WebMvcTest

- MVC를 위한 테스트, 웹에서 테스트하기 힘든 컨트롤러를 테스트하는데 적합
- `요청과 응답`에 대한 테스트 진행하며 내부적으로 동작하는 비즈니스 로직에는 관심을 갖지 않음
- 아래 항목들만 로드되기 때문에 가볍게 테스트 가능
  - `@Controller`
  - `@ControlerAdvice`
  - `@JsonComponent & @Filter`
  - `WebMvcConfigurer`
  - `HandlerMethodArgumentResolver `
- 다른 설정들은 자동으로 올리지 않기 때문에 @Repository, @Resource, @Service, @Component등은 사용 불가
- 그리고 `MockMvc`를 자동을 설정하여 빈으로 등록

```java
@RunWith(SpringRunner.class)
@WebMvcTest(ArticleApiController.class)
public class ArticleApiControllerTest {
    
    @Autowired
    private MockMvc mvc;
    
    @MockBean
    private ArticleService articleService;

    @Test
    public void testGetArticles() throws Exception {
        List<Article> articles = asList(
                new Article(1, "kwseo", "good", "good content", now()),
                new Article(2, "kwseo", "haha", "good haha", now()));

        given(articleService.findFromDB(eq("kwseo"))).willReturn(articles);

        mvc.perform(get("/api/articles?author=kwseo"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("@[*].author", containsInAnyOrder("kwseo", "kwseo")));
    }

    private Timestamp now() {
        return Timestamp.valueOf(LocalDateTime.now());
    }
}
```

- `given` 에서 해당 객체르 생성
- `when` 에서 Mock 객체를 기반으로 미리 정의된 객체를 반환
- `then` 에서 해당 객체의 응답값을 검사



#### MockMvc & ResultActions

> `MockMvc`
>
> http 관련 요청응답을 관장하는 객체

- `perform()`
  - get/post/put/delete
- `andExpect()`
- `andDo()`
  - andDo(print())
- `content(Object requestBody)`
  - body



> `ResultActions`
>
> https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/ResultActions.html

- `andExpect`
- `andDo`
- `andReture`
  - 결과값반환

```java
MvcResult result = actions.andReturn();
```



#### MultipartFile

> MultipartFile이란
>
> 사용자가 업로드한 File을 핸들러에서 손쉽게 다룰 수 있도록 도와주는 Handler 매개변수중 하나



- 파일 업로드 `MockMvc`로 실행시키기
  - `MockHttpServletRequestBuilder` 방식은 `POST` Method
  - 사용자가 업로드한 데이터를 MultipartFile 매개변수에 맵핑하며, upload 요청을 `body parameter`로 전달합니다.

```java
MockMultipartFile mockMultipartFile = new MockMultipartFile(
    "file", "fileName", "text/plain", bytes);

// POST & BodyParameter
MockHttpServletRequestBuilder builder =
    MockMvcRequestBuilders.fileUpload("/apipath")
    .file(mockMultipartFile);

final ResultActions actions = mockMvc
	.perform(builder)
    .andDo(print());
```



### @AutoConfigureMockMvc

@WebMvcTest외에 MVC를 테스트 할 수 있는 다른 방법

위 설정은 MVC 테스트 외의 모든 설정을 같이 올립니다. 실제적으로 동작하는 MVC 테스트 진행이 가능합니다.

@SpringBootTest와 같이 사용이 가능합니다.



### @RestClientTest

- REST관련 테스트를 도와주는 어노테이션
- REST통신의 JSON 형식이 예상대로 응답을 반환하는지 등을 테스트

```java
@RunWith(SpringRunner.class)
@RestClientTest(BookRestService.class)
public class BookRestServiceTest {
    
    @Rule
    public ExpectedException thrown = ExpectedException.none();
    
    @Autowired
    private BookRestService bookRestService;
    
    @Autowired
    private MockRestServiceServer server;
    
    @Test
    public void rest_test() {
        server.expect(requestTo("/rest/test"))
                .andRespond(
                        withSuccess(
                            new ClassPathResource("/test.json", getClass()), 
                            MediaType.APPLICATION_JSON));
        Book book = bookRestService.getRestBook();
        assertThat(book.getId(), is(notNullValue()));
        assertThat(book.getTitle(), is("title"));
        assertThat(book.getPrice(), is(1000D));
    }
}
```

- @RestClientTest는 테스트 대상이 되는 빈을 주입

- `@Rule`로 지정한 필드 값은 `@Before, @After` 어노테이션에 상관없이 하나의 테스트가 메서드가 끝날 때마다 정의한 값으로 초기화 시켜줌

- 테스트에서 자체적으로 규칙을 정의하여 재사용할 때 유용

- `MockRestServiceServer`는 클라이언트와 서버사이의 REST 테스트를 위한 객체입니다.
- 내부에서 RestTempalte을 바인딩하여 실제로 통신이 이루어지게끔 구성 가능
  - 이 코드에서는 Mock 객체와 같이 실제로 통신이 이뤄지지는 않지만 지정된 경로에 예상되는 반환값을 명시




## Data Test

### @DataJpaTest

- JPA 관련된 설정만 로드
- 데이터소스의 설정이 정상적인지 JPA를 사용해서 데이터를 제대로 생성, 수정, 삭제하는지 등 테스트 가능
- 기본적으로 인메모리 DB 이용
- @Entity 클래스를 스캔하여 스프링 데이터 JPA 저장소 구성

```java
@RunWith(SpringRunner.class)
@DataJpaTest
@ActiveProfiles("test")
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
public class BookJpaTest {
    
    @Autowired
    private BookRepository bookRepository;
    
    @Test
    public void book_save_test() {
        final Book book = new Book("title", 1000D);
        final Book saveBook = bookRepository.save(book);
        assertThat(saveBook.getId(), is(notNullValue()));
    }
    
    @Test
    public void book_save_and_find() {
        final Book book = new Book("title", 1000D);
        final Book saveBook = bookRepository.save(book);
        final Book findBook = bookRepository.findById(saveBook.getId()).get();
        assertThat(findBook.getId(), is(notNullValue()));
    }
}
```

- `@AutoConfigureTestDatabase` 어노테이션의 기본 설정값인 `Replace.Any`를 사용하면 기본적으로 내장된 데이터 소스 사용
- 위와 같이 `@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)` 설정시
  `@ActiveProfiles("test")` 기준으로 프로파일 설정
- `@DataJpaTest`는 끝날 때마다 자동으로 데스트에 사용한 데이터를 롤백



### @JsonTest

- JSON의 직렬화 역질렬화를 수행하는 라이브러리인 Gson & Jackson의 테스트를 제공

```java
@RunWith(SpringRunner.class)
@JsonTest
public class BookJsonTest {
    @Autowired
    private JacksonTester<Book> json;
    @Test
    public void json_test() throws IOException {
        final Book book = new Book("title", 1000D);
        String content= "{\n" +
                "  \"id\": 0,\n" +
                "  \"title\": \"title\",\n" +
                "  \"price\": 1000\n" +
                "}";
        assertThat(json.parseObject(content).getTitle()).isEqualTo(book.getTitle());
        assertThat(json.write(book)).isEqualToJson("/test.json");
    }
}
```



### @Transactional

다음 어노테이션을 사용하면 테스트가 끝날 때 rollback 됩니다.

다만 테스트가 서버의 다른 스레드에서 실행중이라면 ( WebEnviroent.RANDOM_PORT, DEFINED_PORT ) 트랜잭션이 롤백되지 않음



## 추가팁

- `@ActiveProfiles("test")` 와 같은 방식으로 원하는 프로파일 환경값 부여 가능





https://meetup.toast.com/posts/124

https://cheese10yun.github.io/spring-boot-test/

https://jojoldu.tistory.com/226
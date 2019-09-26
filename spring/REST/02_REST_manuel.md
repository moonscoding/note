\#spring #REST #setting

## Manuel

### @RestController

- 요청데이터와 응답데이터는 `HttpMessageConmverter`를 통해 가져오고, 반환합니다.
- 입력값 검사에 대한 오류 처리는 예외 핸들러에서 공통으로 수행합니다. 



> 선언형처리 vs 프로그래밍형처리

- 요청데이터를 요청본문형태로 받아낼 인수에 `@RequestBody`를 사용합니다.
- 응답할 데이터를 반환하되, 그 결과가 응답 본문 형태가 되도록 메서드에 `@ResponseBody`를 사용합니다.
  - @Controller 대신 @RestController를 사용하면 생략가능합니다.
- 입력값 검사 결과는 BindingResult로 받아 처리하는 대신 예외로 받아 처리합니다.

```java
@RestController
@RequestMapping("books")
public class BookRestController() {

    // 선언형처리
    @RequestMapping(path="/messages", method=RequestMethod.POST)
    public Message post(@Validated @RequestBody Message newResource) {
        
    }
    
    // 프로그래밍형처리
    @ResponseBody
    public Message post(@Validated @RequestBody Message newResource) {
        Message createdResource = service.create(newMessage);
        return createdResource;
    }
}
```



### API

#### Resource

- `JSON 필드명과 자바빈즈 프로퍼티명을 똑같이 맞춰야 합니다.`

> BookResource

```json
{
    "bookid" : "9791158390747" ,
	"name": "슬랙으로 협업하기",
	"publishedDate" : "2017-08-10"
}
```

```java
public class BookResource implements Serializable {
    private static final long serialVersionUID = 5535788271499457190L;
    private String bookid;
    private String name;
    
    @JsonFormat(pattern="yyyy-MM-dd")
    private java.time.LocalDate publishedDate;
}
```



#### CRUD

- `GET`
  - `요청` curl http://llocalhost:8080/books/9791158390747
    - `응답 (기본)` { "bookld" : "9791158390747" , "name" : "슬랙으로 협업하기 " , "publishedDate" : [2017, 8, 10] }
    - `응답 (@JsonFormat지정)` { "bookld" : "9791158390747" , "name" : "슬랙으로 협업하기 " , "publishedDate" : "2017-08-10" } 
- `POST`
  - `@RequestBody` 로 요청 본문의 데이터 `JSON`을 받습니다.
  - `@Validated`를 이용해서 리소스 객체에 대한 입력값을 검사합니다.
  - 작성한 정보에 접근한기 위한 URI를 생성합니다. URI는 Location 헤더에 설정됩니다.
  - 리소스가 정상적으로 생성됬다는 뜻으로 `201 Created` 를 응답합니다.
    - `ResponseEntity`를 사용하면 응답 헤더를 설정할 수 있습니다.
    - ResponseEntity의 `created()` 메소드를 사용하면 인수에 지정한 URI가 들어가며, 상태코드가 201로 설정됩니다. 
- `PUT`
  - `@ResponseStatus` 를 이용하여 응답할 상태코드를 지정합니다.
    - `204 No Content`는 서버에서 반환할 콘텐츠가 없다는 의미입니다.
    - 만약 갱신된 콘텐츠를 반환받고 싶다면 갱신된 리소스를 반환하며 `200 OK` 로 응답할 수 있습니다.
- `DELETE`
  - `put`과 동일

```java
@Autowired
BookService bookService;

@RequestMapping(path="{bookld}", method=RequestMethod.GET)
public BookResource getBook(@PathVariable String bookld) {
    Book book = bookService.find(bookld);
    BookResource resource = new BookResource();
    resource.setBookld(book.getBookld());
    resource.setName(book.getName());
    resource.setPublishedDate(book.getPublishedDate());
    return resource;
}

@RequestMapping(method=RequestMethod.POST)
public ResponseEntity〈Void〉createBook(@Validated @RequestBody BookResource newResource) {
    Book newBook = new Book();
    newBook.setName(newResource.getName());
    newBook.setPublishedDate(newResource.getPublishedDate());
   
    Book createdBook = bookService.create(newBook);
    String resourceUri = "http://localhost:8080/books/" + createdBook.getBookid();
    return ResponseEntity.created(URI.create(resourceUri)).build();
}

@RequestMapping(path = "{bookld}", method=RequestMethod.PUT)
@ResponseStatus(HttpStatus.NO_CONTENT)
public void put(@PathVariable String bookid, @Validated @RequestBody BookResource resource) {
    Book book = new Book();
    book.setBookld(bookld);
    book.setName(resource.getName());
    book.setPublishedDate(resource.getPublishedDate());
    bookService.update(book);
}

@RequestMapping(path="{bookid}", method=RequestMethod.DELETE)
@ResponseStatus(HttpStatus .NO_CONTENT)
public void delete(@PathVariable String bookid) {
    bookService.delete(book id);
}
```



#### Resource검색

- ID를 대신해서 검색 조건을 요청으로 전송 서버 측에서는 해당 조건을 받아서 처리하는 방식

> BookResourceQuery

```java
public class BookResourceQuery implements Serializable {
    private static final long serialVersionUID = 5535788271499457190L;
    private String name;
    @DateTimeForate(iso=DateTimeFormat.ISO.DATE)
    private LocalDate publishedDate;
}
```

> 검색 REST API 구현

- 검색조건을 담을 수 있는 클래스를 지정합니다.
- 검색조건에 `@Validated`를 사용하여 입력값 검사를 진행합니다.
- 비즈니스로직에 전달하기 위해 POJO인 `BookCriteria`에 요청 파라미터 정보를 옮깁니다.

```java
@RequestMapping(method=RequestMethod.GET)
public BookResource searchBoos(@Validated BookResourceQuery query) {
    BookCriteria criteria = new BookCriteria() ;
    criteria.setName(query.getName());
    criteria.setPublishedDate(query.getPublishedDate());
    List〈Book〉 books = bookService.findAllByCriteria(criteria);

    return books.stream()
        .map(book -〉｛
             BookResource resource = new BookResource();
             resource.setBookid(book.getBookid()) ;
             resource.setName(book.getName()) ;
             resource.setPublishedDate(book.getPublishedDate());
             return resource; })
        .collect(Collectors.tolist());
}
```

> Stream 이용 처리 방식

```java
public List〈Book〉findAllByCriteria(BookCriteria criteria) {
return bookRepository
    .values()
    .stream()
    .filter(book -〉 
            (criteria.getName() = null || book.getName().contains(criteria.getName())) 
            &&
            (criteria.getPublishedDate() = null || book.getPublishedDate().equals(criteria.getPublishedDate())))
    .sorted((o1, o2) -〉 o1.getPublishedDate().compareTo(o2.getPublishedDate()))
    .collect(Collectors.tolist());
```



### CORS

- `CORS ` (Cross-Origin Resource Sharing)
  - AJAX(XMLHttpRequest)를 사용할 때 다른 도메인의 서버리소스에 접근하기 위한 메커니즘
-  지정방법
  - 빈을 정의하는 방식으로 애플리케이션 단위로 설정
  - `@CrossOrigin`을 이용해서 컨트롤러와 핸들러 메서드 단위로 설정

> 리소스접근시 돌아오는 응답헤더

```
Access-Control-Allow-Origin: http://example:8080
Access-Control-Allow-Credentials: true
Vary: Origin
```

> 빈설정

- `WebMvcConfigurerAdapter`클래스의 `addCrosMappings`메서드를 오버라이드 처리
  - `CorsRegistry`클래스의 addMapping 메서드를 이용해서 CORS 기능을 사용할 경로를 지정

```java
@Configuration
@EnableWebMvc
public class WebMvcConfig extends WebMvcConfigurerAdapter {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**");
    }
}
```

> @CrossOrigin

- `@CrossOrigin` 지정시에 모든 핸들러 메서드에 대해 CORS 설정이 적용됩니다.
- 메서드에 `@CrossOrigin`을 지정하여 커스텀할 수 있습니다.

```java
@CrossOrigin
@RequestMapping("books")
@RestController
public class BooksRestController {
    @CrossOrigin(maxAge = 900)
    @RequestMapping(path = "{bookid}", method=RequestMethod.GET)
    public BookResource getBook(@PathVariable String bookld) {
        
    }
}
```



#### CORS옵션


- `allowedOrigins`
  - 접근을 허용할 오리진(도메인) 지정
  - 기본값 : '*' (모두이용가능)
- `allowdMethods`
  - 접근을 허용할 HTTP 메서드 지정
  - 기본값 : '*' (모두이용가능)
- `allowHeaders`
  - 접근을 허용할 헤더를 지정
  - preflight 요청이 들어올때 이 값으로 점검
  - preflight 요청에 대한 응답으로 `Access-Control-Allow-Headers `헤더에 설정
  - 기본값 : '*' (모두이용가능)
- `exposedHeaders`
  - 허용할 데이터의 `화이트리스트(WhiteList)`를 지정
    - `화이트리스트란` 식별된 일부 실체들이 특정 권한, 서비스, 이동, 접근, 인식에 대해 명시적으로 허가하는 목록
  - 지정한 헤더가 응답의 `Access-Control-Expose-Headers `해더에 설정
- `allowCredentials`
  - 인증정보(쿠기나 Basic인증)를 취급할지 여부를 결정
  - true를 지정하면 `Access-Control-Allow-Credentials` 해더에 설정
  - 기본값 : true (인증정보취급)
- `maxAge`
  - 클라이언트가 preflight 요청에 대한 응답을 캐시할 시간(초단위)을 지정
  - preflight 요청에 대한 응답의 `Access-Control-Max-Age` 해더에 설정
  - 기본값 : 1800 (30분)



#### URI조립

- 스프링은 URI를 생성하는 `UriComponentsBuilder` 컴포넌트 제공
- 스프링 MVC에서 핸들러 메서드의 정의 정보와 연동하여 URI를 생성하는 `MvcUriComponentsBuilder` 컴포넌트 제공

##### UriComponentsBuilder

- 프로토콜, 호스트명, 포트번호, 컨텍스트 경로와 같이 환경에 의존하는 부분의 은폐
- URI 템플릿을 사용한 URI 조립

> UriComponentsBuilder 으로 URI 조립하기

- `UriComponentsBuilder` 환경에 의존적이지 않은 값으로 빌드할 수 있습니다.

```java
@RequestMapping(method=RequestMethod.POST)
public ResponseEntity<Void> createBook(
       @Validated @RequestBody BookResource newResource,
       UriComponentsBuilder uriBuilder
) {

   // == URI조립 (UriComponentsBuilder방식) ==
   URI resourceUri = uriBuilder.path("books/{bookId}")
           .buildAndExpand(createBook.getBookId())
           .encode()
           .toUri();
   return ResponseEntity.created(resourceUri).build();
}
```

##### MvcUriComponentBuilder

- `MvcUriComponentBuilder`를 이용하면 핸들러 메서드의 정의정보(요청 매핑이나 메서드 매개변수 정보)와 연동하여 URI를 조립할 수 있어 조금 더 유연합니다.
  - 작성할 URI의 템플릿을 의식할 필요가 없음
  - 타입에 안전

> MvcUriComponentBuilder 으로 URI 조립하기

- `MvcUriComponentsBuilder`의 `relativeTo` 메서드를 사용해서 인수로 받은 UriComponentBuilder를 
  `기본 URL`을 조립하기 위한 객체로 사용
- `on`메서드는 URI를 생성할 때 필요한 컨트롤러의 `mock`객체를 만드는 MvcUriComponentBuilder의 static 메서드
- `on` 메서드로 만들어진 mock 객체의 `@RequestMapping`이 붙은 메서드를 호출하면, 
  그 때 사용된 경로와 경로 변수가 조합된 컨트롤러를 얻을 수 있습니다.
  - mock 객체의 getBook이 호출되며 경로 변수 값을 인자로 받아 `/book/{bookId}`와 같은 경로를 만들어 냅니다.

```java
@RequestMapping(path="{bookId}", method=RequestMethod.GET)
public BookResource getBook(@PathVariable String bookId) {

}

@RequestMapping(method=RequestMethod.POST)
public ResponseEntity<Void> createBook(
    @Validated @RequestBody BookResource newResource,
    UriComponentsBuilder uriBuilder
) {

   // == URI조립 (MvcUriComponentsBuilder방식) ==
   URI resourceUri = MvcUriComponentsBuilder
   	.relativeTo(uriBuilder)
    .withMethodCall(on(BooksRestController.class))
    .getBook(createBook.getBookId()).build().encode().toUri();

   return ResponseEntity.created(resourceUri).build();
}
```
\#spring #resource

---

## Resource

### #구조?

- 스프링이 제공하는 리소스 추상화 기능
- 구체적 위치정보를 직접 다루지 않고 리소스에 접근

> InputStreamSource

```java
public interface InputStreamSource {
  InputStream getInputStream() throws IOException;
}
```

> Resource

- exists() 
  - 리소스 존재 여부확인
- isOpen()
  - 읽기스트림이 열려 있는지 확인

```java
public interface Resource extends InputStreamSource {
  boolean exists();
  boolean isReadable();
  boolean isOpen();
  URL getURL() throws IOException;
  URI getURI() throws IOException;
  File getFile() throws IOException;
  long contentLength() throws IOException;
  long lastModified() throws IOException;
  Resource createRelative(String relativePath) throws IOException;
  String getFilename();
  String getDescription();
}
```

> WritableResource

```java
public interface WritableResource extends Resource {
  boolean isWritable();
  OutputStream getOutputStream() throws IOException;
}
```



### #구현클래스

- `ClassPathResource`
  - 클래스패스 상의 리소스를 다루기 위한 클래스
- `FileSystemResource`
  - java.io 패키지의 클래스를 사용해 파일 시스템상의 리소스를 다루기 위한 클래스
- `PathResource`
  - java.nio.file 패키지의 클래스를 사용해 파일 시스템상의 리소스를 다루기 위한 클래스
- `UrlResource`
  - URL상의 웹 리소스를 다루기 위한 클래스
  - [EX] "http://", "file://"
- `ServletContextResource`
  - 웹애플리케이션 상의 리소스를 다루기 위한 클래스



### #ResourceLoader

- Resource 객체를 생성하는 과정을 추상화하기 위해 제공
- DI 컨테이너를 구성하는 `ApplicationContext`도 다음 `ResourceLoader`의 구현체

> ResourceLoader

- `ResourceLoader`의 구현체는 리소스 경로의 정보를 보고 이에 맞는 `Resource` 의 구현체를 선택

```java
public interface ResourceLoader {
  String CLASSPATH_URL_PREFIX = ResourceUtils.CLASSPATH_URL.PRFIX;
  Resource getResource(String location);
  ClassLoader getClassLoader();
}
```

> ResourcePatternResolver

- 앤트(Ant)형식의 와일드카드를 이용 패턴에 맞는 여러개의 리소스를 관리

```java
public interface ResourcePatternResolver extents ResourceLoader {
  String CLASSPATH_ALL_URL_PREFIX = "classpath*:";
  Resource[] getResources(String locationPattern) throws IOException;
}
```



### #Resource접근

>  `UrlResource` 사용예제

```java
public void accessResource() throws IOException {
  Resource greetingResource = new UrlResource("http://localhost:8080/myApp/greeting.json");
  InputStream in;

  try {
    in = greetingResource.getInputStream();
    String content = StreamUtils.copyToString(in, StandardCharsets.UTF_8);
    System.out.println(content);
  } catch(IOException e) {
    throw new IOException();
  } finally {
    in.close();
  }
}
```

>  `ResourceLoader` 이용주입

- 구현 클래스 의존성 제거
- 프로퍼티를 이용한 의존성 제거

``` java
@Autowried
ResourceLoader resourceLoader;

public void accessResource() throws IOException {
	Resource greetingResource = resourceLoader.getResource("http://localhost:8080/myApp/greeting.json");
}
```

```java
@Value("${resource.greeting:http://localhost:8080/myApp/greeting.json}")
Resource greetingResource;

public void accessResource() throws IOException {
    try (InputStream in = greetingResource.getInputStream()) {
        String content = StreamUtils.copyToString(in, StandardCharsets.UTF_8);
        System.out.println(content);
    }
}
```


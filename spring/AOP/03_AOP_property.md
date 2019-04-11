\#spring #property

---

### #개념

> Property 사용전

```java
@Bean(destroyMethod="close")
DataSource dataSource() {
  BasicDataSource dataSource = new BasicDataSource();
  dataSource.setDriverClassName("org.postgresql.Driver");
  dataSource.setUrl("jdbc:postgresql://localhost:5432/demo");
  dataSource.setUsername("demo");
  dataSource.setPassword("pass");
  dataSource.setDefaultAutoCommit(false);
  return dataSource;
}
```

> Property 사용후 

```bash
datasource.driver-class-name=org.postgresql.Driver
datasource.url=jdbc:postgresql://localhost:5432/demo
datasource.useranme=demo
datasource.password=pass
```

```java
@Bean(destroyMethod="close")
DataSource dataSource(@Value("${datasource.driver-class-name}") String driverClassName,
                      @Value("${datasource.url}") String url,
                      @Value("${datasource.username}") String username,
                      @Value("${datasource.password}") String password) {
  BasicDataSource dataSource = new BasicDataSource();
  dataSource dataSource = new BasicDataSource();
  dataSource.setDriverClassName(driverClassName);
  dataSource.setUrl(url);
  dataSource.setUsername(username);
  dataSource.setPassword(password);
  dataSource.setDefaultAutoCommit(false);
  return dataSource;
}
```



### #설정

- .properties 파일, JVM 시스템 프로퍼티, 환경변수 등등 다양한 프로퍼티 존재
- 중복프로퍼티가 있다면, 
  - `JVM 시스템 > 환경변수 > .properties 파일` 순으로 적용

```java
@Configuration
@PropertySource("classpath:application.properties")
public class AppConfig {

}
```



### # 사용

- `@Value`를 통해 프로퍼티 주입
- `:` 뒤에는 Default 값을 나타냄

```java
@Component
public class Authenticator {
    
  @Value("${failureCountToLock:5}")
  int failureCountToLock;

  public void authenticate(String username, String password) {
      if(failCount >= failureCountToLock) {
          
      }
  }
}
```


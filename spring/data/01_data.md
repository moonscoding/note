\#spring #datasource

---

## DataSource

- `DataSource` 애플리케이션이 데이터베이스에 접근하기 위한 추상화된 연결방식
- 커넥션을 제공



### #설정

> pom.xml

```xml 
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-jdbc</artifactId>
</dependency>
```

> property

```bash
database.driverClassName=org.postgresql.Driver
database.url=jdbc:postgresql://localhost/sample
database.username=demo
database.password=pass
cp.maxTotal=96
cp.maxIdle=16
cp.minIdle=0
cp.maxWaitMillis=60000
```

#### #모듈제공 데이터소스

- `Common DBCP`나 `Tomcat JDBC Connection Pool`과 같이 서드파티가 제공하는 데이터소스
- `DriverManagerDataSource`같이 스프링이 테스트 용도로 제공하는 데이터소스를 빈으로 등록
- `Commons DBCP`와 같은 커넥션풀을 데이터소스로 사용하는 경우

```java
@Configuration
@PropertySource("classpath:jdbc.properties")
public class PoolingDataSourceConfig {
  @Bean(destroyMethod="close")
  public DataSource dataSource(
      @Value("${database.driverClassName}") String driverClassName,
      @Value("${database.url}") String url,
      @Value("${database.username}") String username,
      @Value("${database.password}") String password,
      @Value("${cp.maxTotal}") String maxTotal,
      @Value("${cp.maxIdle}") String maxIdle,
      @Value("${cp.minIdle}") String minIdle,
      @Value("${cp.maxWaitMillis}") String maxWaitMillis) {
    BasicDataSource dataSource = new BasicDataSource();
    dataSource.setDriverClassName(driverClassName);
    dataSource.setUrl(url);
    dataSource.setUsername(username);
    dataSource.setPassword(password);
    dataSource.setDefaultAutoCommit(false);
    dataSource.setMaxTotal(maxTotal);
    dataSource.setMaxIdle(maxIdle);
    dataSource.setMinIdle(minIdle);
    dataSource.setMaxWaitMillis(maxWaitMillis);
    return dataSource;
  }
}
```

#### #서버제공 데이터소스

- `JNDI(Java Naming and Directory Interface)`를 통해 가져와서 사용
- `jdbc/mydb` JNDI명의 커넥션풀이 있다는 가정

```java
@Configuration
public class JndiDataSourceConfig {
  @Bean
  public DataSource dataSource() {
    JndiTemplate jndiTemplate = new JndiTemplate();
    return jndiTemplate.lookup("java.comp/env/jdbc/mydb", DataSource.class);
  }
}
```

#### #내장 데이터소스

- `HSQLDB, H2, Apache Derby` 등등 프로토타입을 위해 사용
- 애플리케이션 구동시 DB가 새로 구축되므로 `DDL, DML`이 추가적으로 필요

```java
@Configuration
public class DataSourceEmbeddedConfig {
    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .setScriptEncoding("UTF-8")
            .addScript("META-INF/sql/schema.sql", "META-INF/sql/insert-init-data.sql")
            .build();
    }
}
```


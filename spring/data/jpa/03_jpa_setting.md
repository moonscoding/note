\#spring #JPA #setting



### #MavenDependency

```xml
<!-- Spring data JPA -->
<dependency>
   <groupId>org.springframework.data</groupId>
   <artifactId>spring-data-jpa</artifactId>
</dependency>

<!-- hibernate core -->
<dependency>
   <groupId>org.hibernate</groupId>
   <artifactId>hibernate-core</artifactId>
</dependency>

<!-- hibernate entity-manager -->
<dependency>
   <groupId>org.hibernate</groupId>
   <artifactId>hibernate-entityManager</artifactId>
</dependency>
```



### #DataSource

> EntityManagerFactory

```java
@Configuration
public class JpaConfig {

  @Autowired
  private DataSource dataSource;

  @Bean
  public JpaVendorAdapter jpaVendorAdapter() {
      HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
      vendorAdapter.setDatabase(Database.H2);
      vendorAdapter.setShowSql(true);
      return vendorAdapter;
  }

  @Bean
  public LocalContainerEntityManagerFactoryBean entityManagerFactoryBean(DataSource dataSource) {
      LocalContainerEntityManagerFactoryBean factory = new LocalContainerEntityManagerFactoryBean();
      factory.setDataSource(dataSource);
      factory.setPackagesToScan("example.domain.model");
      factory.setJpaVendorAdapter(jpaVendorAdapter());
      return factory;
  }
}
```

- `HibernateJpaVendorAdapter`
  - JPA 구현의 독자적인 설정을 하기위해 `JpaVendorAdapter` 인터페이스 구현한 클래스의 빈을 정의
  - 하이버네이트를 사용하기때문에 하이버네이트 전용 구현클래스인 HibernateJpaVendorAdapter 사용
- `setDatabase()`
  - 사용할 DB 제품 설정
- `setShowSql()`
  - SQL을 콘솔에 출력하는 기능 활성화
  - 하이버네이트가 어떤 SQL을 구성하는지 확인할떄 사용
- `LocalContainerEntityManagerFactoryBean`
  - `EntityManagerFactory`가 빈으로써 DI 컨테이너에서 관리
  - 기본상태에선 스프링 데이터 JPA가 `entityManagerFactoryBean`라는 이름을 사용하기 때문에 주의필요
- `setDataSource()`
  - JPA 영속성처리에서 사용하는 데이터소스 설정
- `setPackagesToScan()`
  - Entity 클래스가 정의된 패키지 설정
  - 바로 아래있는 Entity 클래스만 EntityManager로 취급



### #JpaTransactionManager

- Jpa용 트랜잭션관리
- 다음 설정으로 `@Transactional` 을 지정하는 것만으로 JPA 트랜잭션 관리가 가능

```java
@Configuration
@EnableTransactionManagement
public class JpaConfig {

  @Bean
  public PlatformTransactionManager transactionManager(EntityManagerFactory entityManagerFactory) {
      JpaTransactionManager jpaTransactionManager = new JpaTransactionManager();
      jpaTransactionManager.setEntityManagerFactory(entityManagerFactory);
      return jpaTransactionManager;
  }
}
```



### #JPA 활성화

- `@EnableJpaRepository`
  - JPA 초기설정
  - Repository 인터페이스나 커스텀 Repository 클래스가 저장되 있는 패키지명

```java
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories("example.domain.repository")
public class JpaConfig {

}
```



### #OpenEntityManagerInView

- `Lazy 패치`
  - JPA에는 Entity 데이터가 필요할때까지 DB 접근을 수행하지 않음
  - 불필요한 데이터를 DB에서 가져오는 것을 피함
  - 트랜잭션이 종료되고 분리상태로 된 Entity는 Lazy 패치할 수 없어 예상한 값을 취득할 수 없는 문제발생
- `Open Entity Manager In View 패턴`
  - 웹어플리케이션에서는 화면을 랜더링하기 전에 트랜잭션을 종료해 버리는 패턴이 일반적이며,
    화면을 랜더링할 때 Entity에 접근하면 문제가 발생
  - Lazy 패치를 웹화면을 랜더링할 때 사용할 수 있음
  - 트랜잭션이 끝난 후에  EntityManager를 닫지 않고 Entity 관리상태로 유지
  - 웹화면의 랜더링이 끝날때까지 Lazy 패치가능



> WebRequestInterceptor & ServletFilter

- 스프링에서 제공하는 클래스
  - `OpenEntityManagerInViewInterceptor`
    - 유연성높음
  - `OpenEntityManagerInViewFilter`
    - 연장기간이김
    - ServletFilter에서도 Lazy 패치가능
    - 대부분 `OpenEntityManagerInViewInterceptor`에서 충분한 기간을 얻을 수 있음

```java
@Configuration
public class WebMvcConfig extends WebMvcConfigurerAdapter {

  @Bean
  public OpenEntityManagerInViewInterceptor openEntityManagerInViewInterceptor() {
    return new OpenEntityManagerInViewInterceptor();
  }

  @Override
  public void addInterceptors(InterceptorRegistry registry) {
    registry.addWebRequestInterceptor(openEntityManagerInViewInterceptor())
      .addPathPatterns("/**")
      .excludePathPatterns("/**/*.html")
      .excludePathPatterns("/**/*.js")
      .excludePathPatterns("/**/*.css")
      .excludePathPatterns("/**/*.png");
  }
}
```


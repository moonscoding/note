\#MSA #security



## 보안

- 사용자를 적절히 통제

- 인증과 인가
  - 인증(Authentication)
    - 자격증명을 제공해서 자신이 누구인지 증명하는 행위
  - 인가(Authorization)
    - 사용자가 수행하려는 작업의 허용여부를 결정

## OAuth2

- 토큰기반의 보안 프레임워크
  - 사용자는 자원에 접근하려는 app을 통해서 자격 증명을 제시하고 OAuth2 서버에서 인증
  - 사용자의 자격 증명이 유효하면 
    OAuth2 서버는 사용자 app이 이용하는 서비스가 보호 자원(MSA)에 접근하려고 시도할 때마다 제시할 토큰을 제공
  - 보호자원은 OAuth2서버에 접속해서 토큰 유효성을 확인하고 사용자가 지정한 역할을 조회할 수 있음
  - 역할은 연관된 사용자를 함께 그룹으로 묶고 사용자 그룹이 액세스할 수 있는 자원을 정의
- 구성요소
  - `보호자원`
    - 보호하려는 자원(여기서는 MSA)이며 적절한 권한 부여받은 인증된 사용자만 액세스 가능
  - `자원소유자`
    - 서비스를 호출할 수 있는 app 및 서비스에 접근할 수 있는 사용자
    - 서비스에서 수행할 수 있는 작업을 정의
    - 자원 소유자가 등록한 app은 식별가능한 app이름과 시크릿 키를 받음
    - app 이름과 시크릿 키는 OAuth2
  - `애플리케이션`
    -  사용자 대신 서비스를 호출할 app
    - 사용자는 서비스를 직접 호출하지 않는 대신 app에 의존해 작업 수행
  - `OAuth2 인증서버`
    - app과 소비되는 서비스 사이의 중개자
    - app이 사용자 대신 호출하는 모든 서비스에 사용자의 자격 증명을 전달하지 않고 사용자 자신을 인증할 수 있음



### #스프링_OAuth2

- OAuth2의 인증과 인가설정방법을 이해하기 위해서 OAuth2 패스워드 그랜트 타입을 구현
  - 스프링 클라우드 기반 OAuth2 인증 서비스를 설정
  - OAuth2 서비스와 사용자 신원을 인증 및 인가할 수 있도록 인가된 app 역할을 하는 가짜 EagleEye UI app 등록
  - OAuth2 패스워드 그랜트 타입을 사용해서 EagleEye 서비스 보호
    - EagleEye UI를 만들지 않고 EagleEye OAuth2 서비스에 인증하는 PostMan으로 사용자 로그인 시뮬레이션 진행
  - 인증된 사용자만 호출 할 수 있도록 라이선싱 및 조직서비스 보호



### #EagleEye_OAuth2

- 인증서비스는 사용자 자격증명을 인증하고 토큰을 발행
- 인증 서비스가 보호하는 서비스에 사용자가 접근할 시 자기가 발급한 OAuth2 토큰인지 만료되지 않았는지 확인



> pom.xml

- spring-cloud-security
  - 일반적인 스프링과 스프링 클라우드 보안 라이브러리
- spring-security-oauth2
  -  OAuth2 라이브러리를 가져옵니다.

```&gt; xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-security</artifactId>
</dependency>

<!-- <dependency>
    <groupId>org.springframework.security.oauth</groupId>
    <artifactId>spring-security-oauth2</artifactId>
</dependency> -->
```

> Application.java (authentication-service)

- @EnableAuthorizationServer
  - 스프링 클라우드에 서비스가 OAuth2 서비스로 사용되며 OAuth2 인증 및 인가 과정에서 사용될 여러 REST 기반 엔드포인트를 추가할 것이라 알림
- /user(/auth/user로 매핑)
  - OAuth2로 보호되는 서비스에 접근하려할 때 사용
  - 보호 서비스로 호출되어 OAuth2 엑세스 토큰의 유효성을 검증하고 보호 서비스에 접근하는 할당된 사용자 역할을 조회

```java
@SpringBootApplication
@RestController
@EnableResourceServer
@EnableAuthorizationServer // 이 서비스가 OAuth2 서비스가 될 것이라고 스프링 클라우드에게 알림
public class Application {
    
    // 사용자 정보를 조회하는데 사용
    @RequestMapping(value = { "/user" }, produces = "application/json")
    public Map<String, Object> user(OAuth2Authentication user) {
        Map<String, Object> userInfo = new HashMap<>();
        userInfo.put("user", user.getUserAuthentication().getPrincipal());
        userInfo.put("authorities", AuthorityUtils.authorityListToSet(user.getUserAuthentication().getAuthorities()));
        return userInfo;
    }


    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```



### #APP_OAuth2_등록

> OAuth2Config.java

```java
@Configuration 
public class OAuth2Config extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private AuthenticationManager authenticationManager;

    @Autowired
    private UserDetailsService userDetailsService;

    // configure() 재정의
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
                .withClient("eagleeye")
                .secret(PasswordEncoderFactories.createDelegatingPasswordEncoder().encode("thisissecret"))
                .authorizedGrantTypes("refresh_token", "password", "client_credentials")
                .scopes("webclient", "mobileclient");
    }

    // AuthorizationServerConfigurerAdapter 안에서 사용될 여러 컴포넌트를 정의
    // 스프링의 기본 인증 관리자와 스프링과 함께 제공되는 사용자 상세 서비스를 이용한다고 알려줌
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
      endpoints
        .authenticationManager(authenticationManager)
        .userDetailsService(userDetailsService);
    }
}
```




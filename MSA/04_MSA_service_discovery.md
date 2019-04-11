\#MSA #serviceDiscovery



## 서비스 디스커버리

- 서비스 위치 찾기
- 클라우드에서 서비스 디스커버리
- c스프링 유레카 서비스 구축
- 스프링 유레카에 서비스 등록
- 서비스 디스커버리를 사용한 서비스 검색



![1552636395720](1552636395720.png)

### #유레카

- 서비스 디스커버리 에이전트를 설정
- 여러 서비스 에이전트를 등록해서 서비스 디스커버리를 구현
- 서비스 디스커버리에서 얻은 정보를 이용해서 다른 서비스를 호출할 수 있음



#### #과정

- 서비스 부트스트래핑 시점에 라이선싱 및 조직 서비스는 자신의 유레카 서비스에 등록
  - 서비스 ID와 함께 각각 서비스 인스턴스의 물리적 위치/포트 번호를 유레카에 알려줌
- 라이선싱 서비스가 조직 서비스를 호출할 때 넥플릭스 리본 라이브러리를 사용해서 클라 측 부하 분산 기능 수행
  - 리본 라이브러리는 유레카 서비스에 서비스 위치 정보를 조회하고 로컬 캐싱
- 주기적으로 넥플릭스 리본 라이브러리는 유레카 서비스를 핑해서 로컬 캐시의 서비스 위치를 새로고침



#### #구축

- server.port
- registerWithEureka
- fetchRegistry
- waitTimeInMsWhenSyncEmpty
  - 로컬에 유레카 서비스를 테스트할 시에 이 프로퍼티가 없으면 등록한 서비스를 즉시 알리지 않음 (주석제거필요)
  - 기본적으로 5분을 기다린 후 등록된 서비스 정보를 공유
  - 유레카는 등록된 서비스에서 10초 간격으로 연속 3회 상태 저보를 받아야 함으로 등록된 개별 서비스를 보여준는데 30초 소요

> application.yml (Eureka)

```yaml
# Default port is 8761
server:
  port: 8761 # 유레카 서버가 수신 대기할 포트

eureka:
  client:
    registerWithEureka: false # 유레카 서비스에 (자신을) 등록하지 않음
    fetchRegistry: false # 레지스트리 정보를 로컬에 캐싱하지 않음
  server:
    waitTimeInMsWhenSyncEmpty: 5 # 서버가 요청을 받기 전 대기할 초기 시간
  serviceUrl:
    defaultZone: http://localhost:8761
```



### #서비스등록

- 조직과 라이선싱 서비스를 구성해 유레카 서비스에 등록

> pom.xml (Client)

- spring-cloud-starter-netflix-eureka-client
  - 스프링 클라우드가 유레카 서비스와 상호 작용하는데 필요한 jar 파일 포함

```xml 
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

> bootstrap.yml (Client)

- 유레카 서비스에 등록하는 서비스는 `애플리케이션  ID`와 `인스턴스 ID`라는 두 가지 구성 요소 필요
  - `애플리케이션 ID`는 서비스 인스턴스 그룹을 의미 - spring.application.name 프로퍼티값으로 설정
  - `인스턴스 ID`는 개별 서비스 인스턴스를 인식하는 임의의 숫자

```
spring:
  application:
    name: organizationservice
  profiles:
    active:
      default
  cloud:
    config:
      enabled: true
```

> application.yml (Client)

- 유레카 서비스에 서비스를 등록하는 위치와 방법
  - `eureka.instance.preferIpAddress`프로퍼티는 서비스의 호스트 이름이 아닌 IP 주소를 유레카에 등록 
  - `eureka.client.registerWithEureka` 프로퍼티는 조직 서비스 자신을 유레카 서비스에 등록하도록 지정

```yaml 
eureka:
  instance:
    preferIpAddress: true # 서비스 이름 대신 서비스 IP 주소 등록
  client:
    registerWithEureka: true # 유레카에 서비스 등록
    fetchRegistry: true # 유레카 서비스 위치
    serviceUrl:
        defaultZone: http://localhost:8761/eureka/
```



#### #반환형식

- `eureka` 서비스가 반환하는 기본 형식은 XML
- HTTP Accept 헤더를 application/json 으로 설정하면 json 반환도 가능합니다.



#### #대기시간(warn-up)

- `eureka`에 서비스가 등록되면 가용하다고 확인될 때까지 30초간 연속 세 번의 상태 정보를 확인하며 대기



### #서비스검색

- 스프링 디스커버리 클라이언트
- RestTemplate 활성화된 스프링 디스커버리 클라이언트
- 넥플릭스 Feign 클라이언트



#### #DiscoveryClient

> LicenseServiceController.java

```java
@RequestMapping(value="/{licenseId}/{clientType}",method = RequestMethod.GET)
public License getLicensesWithClient( @PathVariable("organizationId") String organizationId,
                                     @PathVariable("licenseId") String licenseId,
                                     @PathVariable("clientType") String clientType) {
    return licenseService.getLicense(organizationId,licenseId, clientType);
}
```

- Discovery 
  - 디스커버리 클라이언트와 표준 스프링 RestTemplate 클래스를 사용해서 조직 서비스 호출
- Rest
  - 향상된 스프링 RestTemplate을 사용해서 리본 기반의 서비스 호출
- Feign
  - 넷플릭스 Feign 클라이언트를 사용해서 리본을 통해 서비스 호출

> LicenseService.java

```java
public License getLicense(String organizationId,String licenseId, String clientType) {
    License license = licenseRepository.findByOrganizationIdAndLicenseId(organizationId, licenseId);

    Organization org = retrieveOrgInfo(organizationId, clientType);

    return license
        .withOrganizationName( org.getName())
        .withContactName( org.getContactName())
        .withContactEmail( org.getContactEmail() )
        .withContactPhone( org.getContactPhone() )
        .withComment(config.getExampleProperty());
}
```

- `DiscoveryClient`와 현실
  - DiscoveryClient를 살펴보며 리본으로 서비스 소비자를 구축하는 방법을 마무리
  - 실제 서비스가 리본에 질의해서 등록된 서비스와 서비스 인스턴스를 알아야 할 때만  직접 DiscoveryClient 사용
    - 문제점
      - 리본 클라이언트 측 부하 분산의 장점을 얻지 못함
      - 너무 많은 일을 처리



#### #RestTemplate

- 리본을 지원하는 RestTemplate을 사용하는 방법을 보여주는 예제
- 리본 지원 RestTemplate 클래스를 사용하려면 `@LoadBalanced` 애너테이션으로 RestTemplate 빈 생성 메서드를 정의

```java
@LoadBalanced
@Bean
public RestTemplate getRestTemplate(){
    return new RestTemplate();
}
```



#### #Feign

- 넥플릿스 Feign 클라이언트로 서비스 호출
  - 스프링 리본이 활성화된 RestTemplate 클래스에 다른 대안
  - 개발자가 자바 인터페이스를 먼저 정의한 후 리본이 호출할 유레카 기반의 서비스를 매핑하고 
  - 그 인터페이스 안에 스프링 클라우드 애너테이션을 추가해서 REST 서비스를 호출하는 접근

> Application.java

```java
@SpringBootApplication
@EnableDiscoveryClient // FeignClient만 사용함으로 @EnableDiscoveryClient 제거 가능
@EnableFeignClients // FeignClient 사용을 위해 @EnableFeignClients 추가

public class Application {

  // ..

}
```

> OrganizationFeignClient.java

- @FeignClient 애너테이션 사용해서 인터페이스를 대표할 서비스 애플리케이션 ID 전달
- getOrganization() 로 클라이언트가 조직 서비스를 호출

```java
@FeignClient("organizationservice")
public interface OrganizationFeignClient {
    @RequestMapping(
            method= RequestMethod.GET,
            value="/v1/organizations/{organizationId}",
            consumes="application/json")
    Organization getOrganization(@PathVariable("organizationId") String organizationId);
}
```



### #요약

- 서비스 디스커버리 패턴은 서비스의 물리적 위치 추상화에 사용
- 유레카 같은 서비스 디스커버리 엔진은 서비스 클라이언트에 영향을 주지 않고 해당 환경의 서비스 인스턴스를 원활하게 추가/삭제
- 클라이언트 측 부하 분산을 사용하면 서비스 호출하는 클라이언트에서 서비스의 물리적 위치를 캐싱해 더 나은 성능 및 회복성 제공
- 유레카는 넥플릭스 프로젝트의 스프링 클라우드와 사용하면 쉽게 구축하고 구성 가능
- 스프링 클라우드와 넥플릭스 유레카 그리고 서비스를 호출하는 넥플릭스 리본으로 다음 세가지 메커니즘 사용
  - 스프링 쿨라우드와 DiscoveryClient
  - 스프링 클라우드와 리본 지원 RestTemplate
  - 스프링 클라우드와 넥플릭스 Feign 클라이언트

# 
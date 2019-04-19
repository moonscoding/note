\#MSA #recovery



## 클라이언트 회복성패턴

- 분산시스템에서 장애를 처리하기 위한 방법
  - 서비스 저하는 간헐적으로 발생하고 확산될 수 있음
  - 원격 서비스 호출은 대게 동기식이며 오래 걸리는 호출을 중단하지 않음
  - 애플리케이션은 대게 부분적인 저하가 아닌 원격 자원의 완전한 장애를 처리하도록 설계



### # 회복성패턴

- 클라이언트의 회복성을 위한 소프트웨어 패턴은 원격 서비스가 에러를 던지거나 제대로 동작하지 못해  접그 실패시
- 원격자원을 호출하는 클라이언트 충돌을 막는데 초점이 맞춰짐
- 이 패턴의 목적은 데이터 베이스 커넥션 및 스레드 풀 같이 소중한 클라이언트의 소비자에게 상향 전파되는 것을 막음



#### #종류 

- 클라이언트 측 부하 분산
  - 서비스 클라이언트는 서비스 디스커버리에서 조회한 MSA의 엔드포인트를 캐싱
- 회로 차단기
  - 서비스 클라이언트가 장애중인 서비스를 반복적으로 호출하지 못하게 한다
- 폴백
  - 호출이 실패하면 폴백은 실행 가능한 대안이 있는지 확인
- 벌크헤드
  - 불량 서비스가 클라이언트의 모든 자원을 고갈시키지 않도록 서비스 클라이언트가 수행하는 서비스 호출을 격리



##### #클라이언트_부하분산

- 클라이언트 측 부하 분산은 클라이언트가 넥플릭스 유레카 같은 서비스 디스커버리 에이전트를 이용해 서비스의 모든 인스턴스를 검색한 후 해당 서비스 인스턴스의 실제 위치를 캐싱하는 것
- 서비스 소비자가 서비스 인스턴스를 호출해야 할 때마다 클라이언트 측 로드 밸런서는 서비스 위치 풀에서 관리하는 서비스 위치를 하나씩 전달
- 클라이언트 측 로드 밸런서는 서비스 클라이언트와 서비스 소비자 사이에 위치함으로 서비스 인스턴스가 에러를 전달하거나 불량 동작하지 않는지 감지
- 클라이언트 측 로드 밸런서가 문제를 감지할 수 있다면 가용 서비스 위치 풀에서 문제가 된 서비스 인스턴스를 제거해 서비스 호출이 그 인스턴스로 전달 되는 것을 방지



##### #회로차단기

- 원격 서비스 호출을 모니터링
- 호출이 오래 걸릴경우 회로 차단기가 중재해서 호출을 중단
- 회로 차단기는 원격 자원에 대한 모든 호출을 모니터링하며 호출이 필요한 만큼 실패하면 회로 차단기가 횔성화되어 빨리 실패하게 만듬
- 고장 난 원격 자원은 더 이상 호출되지 않도록 차단



##### #폴백처리

- 원격 서비스에 대한 호출이 실패할 때 예외를 발생시키지 않고 서비스 소비자가 대체 코드 경로를 실행해 다른 방법으로 작업을 수행
- 일반적으로 이 패턴은 다른 데이터 소스에서 데이터를 찾거나 향후 처리를 위해 사용자 요청을 큐에 입력하는 작업과 연관
- 사용자의 호출에 문제가 있다고 예외를 표시하지 않지만 나중에 해당 요청



##### #벌크헤드

- 벌크헤드 패턴을 적용하면 원격 자원에 대한 호출을 자원별 스레드 풀로 불리함으로 특정 원격 자원의 호출이 느려저 전체 애플리케이션이 다운되는 위험을 줄일 수 있음
- 스레드 풀은 서비스를 위한 벌크헤드(격벽) 역할
- 한 서비스가 느리게 반응한다면 해당 서비스 호출을 위한 스레드 풀은 포화되어 요청을 처리하지 못하겠지만 다른 스레드 풀에 할당된 다른 서비스 호출은 포화되지 않음



## #히스트릭트

- 스프링 클라우드와 히스트릭스 랩퍼를 추가하도록 라이선싱 서비스의 메이븐 빌드 파일을 구성
- 스프링 클라우드와 히스트릭스 애너테이션을 사용해서 회로 차단기 패턴으로 원격 호출을 감싼다
- 호출별 타임아웃 설정을 위해 원격 자원에 개별 회로 차단기를 사용자 정의하는 방법을 알아봄
  - 회로 차단기의 구성 방법도 보여주며, 회로 차단기가 작동하기 전에 발생할 실패 횟수도 조절
- 회로 차단기가 호출을 중단하거나 호출이 실패한 경우 폴백 전략을 구현
- 서비스 내 개별 스레드 풀을 사용해서 서비스 호출을 격리하며, 호출되는 원격 자원간에 벌크헤드 구축



> pom.xml (licensing-service)

- spring-cloud-starter-netflix-hystrix

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

> Application.java (licensing-service)

```java
@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker // 스프링 클라우드에 이 서비스에서 히스트릭스를 사용할 것이라고 지정
public class Application {
    @LoadBalanced
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```



### #회로차단기

- 구현방법
  - 라이선싱 및  조직 서비스 모두 자기 데이터베이스에 대한 호출을 히스트릭스 회로 차단기에 연결
  - 두 서비스 사이의 호출을 히스트릭스에 연결

> LicensingService.java (licensing-service)

- `@HystrixCommand`
  -  getOrganization() 메서드 호출마다 히스트릭스 회로 차단기와 해당 호출이 연결
  - 회로 차단기는 메소드 호출이 1,000 미리초보다 오래 걸릴 때마다 호출을 중단

```java
@HystrixCommand
private Organization getOrganization(String organizationId) {
    return organizationRestClient.getOrganization(organizationId);
}
```



- 조직 마이크로서비스에 대한 호출 타임아웃
  - 회로 차단기 동작을 호출 단위로 붙이는 메서드 수준의 애너테이션
    - 데이터베이스에 엑세스하거나 마이크로 서비스를 호출하는데 동일한 애너테이션을 사용하는것
  - 라이선싱서비스에서 라이선스와 연관된 조직 이름을 조회해야 한다고 가정했을때
    - 조직 서비스에 대한 호출을 회로 차단기로 감싸려고 하면, 
      RestTemplate 호출 자체 메소드로 분해하고 @HystrixCommand 만 추가

```java
@HystrixCommand
private Organization getOrganization(String organizationId) {
    return organizationRestClient.getOrganization(organizationId);
}
```

- `@HystrixCommand`는 사용은 쉬우나, 애너테이션 설정 없이 기본형태를 사용함은 주의가 필요
  - 기본적으로 프로퍼티없이 애너테이션을 지정하면 애너테이션은 모든 원격 서비스 호출에 동일한 스레드 풀을 사용하여 애플리케이션에 문제를 발생시킬 수 있음



- 회로 차단기의 타임아웃 사용자 정의

```java

```



#### #폴백_프로세싱

- 회로차단기 패턴은 원격 자원의 소비자와 리소스 사이 `중간자`를 두어 개발자에게 서비스 실패를 가로채며, 
  다른 대안을 선택할 기회를 주는것
- 이것은 히스트릭스에서 폴백전략이라고 알려져 있고, 구현이 쉽다.
- 현재 라이선싱 정보가 가용하지 않다고 알려주는 라이선싱 객체를 만들어 반환하는 
  라이선싱 DB에 대해 간단한 폴백전략을 수립해보자

```java
// fallbackMethod 속성으로 히스트릭스에서 호출이 실패할 때 불러오는 클래스 함수를 정의
@HystrixCommand(fallbackMethod="buildFallbackLicenseList") 
public List<License> getLicensesByOrg(String organizationId){
    logger.debug("LicenseService.getLicensesByOrg  Correlation id: {}", UserContextHolder.getContext().getCorrelationId());
    randomlyRunLong();

    return licenseRepository.findByOrganizationId(organizationId);
}

// fallbackMethod - 이 폴백 메서드에서 하드 코딩된 값을 반환
private List<License> buildFallbackLicenseList(String organizationId) { 
    List<License> fallbackList = new ArrayList<>();
    License license = new License()
        .withId("0000000-00-00000")
        .withOrganizationId( organizationId )
        .withProductName("Sorry no licensing information currently available");

    fallbackList.add(license);
    return fallbackList;
}
```

- 호출이 너무 오래 걸력 히스트릭스를 차단할 때 호출할 메소드 이름을 이 속성에 추가
- 폴백 메서드를 정의
  - 폴백 메서드는 `@HystrixCommand`가 보호하려는 메서드와 같은 클래스에 있어야 합니다.
  - `@HystrixCommand`가 보호하려는 메서드에 전달되는 모든 매개변수를 폴백 메서드가 받음
    - 폴백 세머스는 이전 메서드와 서식이 완전히 동일해야 합니다.



#### #폴백전략

- 폴백전략은 MSA가 데이터를 검색한 호출이 실패하는 상황에 적합하다
  - 폴백은 자원이 타임아웃되거나 실패할 때 행동 방침을 제공하는 메커니즘
  - 폴백을 사용해서 타임아웃 에외를 잡아내고 에러 로깅만 한다면 서비스 호출 전후로 표준 try-catch 블록을 사용하고 로깅 조직을 그 블록 안에 넣어도 무방
  - 폴백 기능으로 수행하는 행동을 알고 있어야 폴백 서비스에서 다른 분산 서비스를 호출한다면 @HystrixCommand로 폴백을 감싸야 할 수 있음
  - 1차 폴백 행동 방침을 겪게 한 동일한 장애가 2차 폴백 옵션에도 영향을 줄 수 있음



### #벌크헤드

- MSA 기반의 애플리케이션이 종종 특정 작업을 완료하기 위해 MSA 호출 필요
- 벌크헤드 패턴을 적용하지 않는다면 기본적인 호출 행위는 전체 자바 컨테이너에 대한 요청을 처리하는 스레드에서 이루어짐
- 대류모 상황에서 한 서비스에 발생한 성능 문제로 자바 컨테이너의 모든 스레드가 최대치에 도달해 작업 처리를 대기하고, 새로운 요청들은 적재
  결국 자바  컨테이너는 비정상 종료
- 히스트릭스는 스레드 풀을 사용해서 원격 서비스에 대한 모든 요청을 위임
- 기본적으로 모든 히스트릭스 명령은 요청을 처리하기 위해 동일한 스레드 풀을 공유
- 이 스레드 풀에는 원격 서비스 호출을 처리할 10개의 스레드가 있고 원격 서비스 호출은 REST 서비스 호출 데이터베이스 호출 등 무엇이든 가능
- 벌크헤드패턴
  - 애플리케이션 안에서 엑세스하는 원격 자원이 적고 비교적 균등하게 각 서비스를 호출 할 때 효과적으로 동작
  - 다른 서비스보다 훨씬 호출량이 많고 호출을 완료하는데 오래걸리는 서비스가 
    히스트릭스 기본 스레드 풀에 있는 모든 스레드를 차지 결국 모든 스레드 고갈
    - 뷴랃한 수래두 퓰울 구현하여 해당 문제를 해결 할 수 있음



```java
@HystrixCommand(fallbackMethod = "buildFallbackLicenseList",
    threadPoolKey = "licenseByOrgThreadPool", // 스레드 풀의 고유 이름을 정의
    threadPoolProperties = { // threadPool 동작을 정의하고 설정
        @HystrixProperty(name = "coreSize",value="30"), // coreSize 속성은 스레드 풀의 스레드 개수를 정의
        @HystrixProperty(name="maxQueueSize", value="10") // maxQueueSize 속성은 스레드 풀 앞에 배치할 큐와 큐에 넣을 요청 수를 정의
    })
public List<License> getLicensesByOrg(String organizationId){
    return licenseRepository.findByOrganizationId(organizationId);
}
```

- `threadPoolKey` 
  - 새로운 스레드 풀을 설정하라고 알림
- `threadPoolProperties`
  - HistrixProperty 객체 배열에 저장하고 HistrixProperty  객체는 스레드 풀의 동작을 제어하는데 사용	
  - `coreSize`
  - `maxQueueSize`
    - 값을 -1로 설정하면 유입된 호출을 유지하는데 자바의 SynchronousQueue가 사용
      - 동기식 큐를 사용하면 본질적으로 스레드 풀에서 가용한 스레드 개수보다 더 많은 요청을 처리 불가
    - 값을 1 이상으로 설정하면 히스트릭스는 자바의 LinkedBlockingQueue를 사용
      - 모든 스레드가 요청을 처리하는데 분주하더라도 더 많은 요청을 큐에 넣을 수 있음
    - 스레드 풀이 처음 초기화 될 때만 설정할 수 있다
      - queueSizeRejectionThreshold 속성을 사용하면 히스트릭스는 큐 크기를 동적으로 변경할 수 있지만, maxQueueSize 속성이 0보다 클 때만 이 속성 설정 가능

> 스레드 풀의 적정 크기는 얼마일까 ( 넥슬릭스 제안 )

- (서비스가 정상일 때 최고점에서 초당 요청 수 x 99 백분위 수 지연 시간(단위:초)) + 오버헤드를 대비한 소량의 추가 스레드



### #세부설정

- 히스트릭스 회로차단기의 동작을 실제로 사용자가 정의하는 방법
- 
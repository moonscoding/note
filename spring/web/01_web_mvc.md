\#spring #MVC

## MVC

- `Model, View, Controller` 로 구성된 웹 애플리케이션 개발 패턴
  - `Model ` 애플리케이션 상태(데이터)나 비즈니스 로직을 제공하는 컴포넌트
  - `View`  모델이 보유한 애플리케이션 상태(데이터)를 참조하고 클라이언트에 반환할 응답 데이터를 생성하는 컴포넌트
  - `Controller` 요청을 받아 모델과 뷰의 호출을 제어하는 컴포넌트, 요청과 응답의 처리 흐름을 제어



### #특징

#### #웹애플리케이션

- `POJO 구현`
  - 컨트롤러 모델등의 클래스를 POJO 형태로 구현
  - 특정 프레임워크에 종속적이지 않아 단위테스트 유리
- `애너테이션을 이용한 설정`
  - 각종 설정정보를 애너테이션으로 정의
- `유연한 메서드 시그니처 정의`
  - 컨트롤러 클래스의 메서드 매개변수에 처리에 필요한 것만 골라서 정의
  - 인수에 지정할 수 있는 타입도 다양
  - 프레임워크가 인수에 전달하는 값을 자동으로 담아주거나 변환 ( 변경 및 리펙토링에 유리 )
- `Servlet API 추상화`
  - 서블릿 API (HttpServletRequest, HttpServletResponse, HttpSession .. 등등) 추상화 기능 제공
  - 컨트롤러 클래스 구현에서 서블릿 API를 직접 사용하는 코드가 제거되어 상대적으로 쉬움
- `뷰 구현 기술의 추상화`
  - 컨트롤러가 뷰 이름을 반환하며, 결과로 뷰 이름에 해당하는 화면이 전달
  - 컨트롤러는 뷰의 이름만 알면 되기 때문에 뷰 기술을 알지 못해도 됩니다. ( 의존성분리 )
- `스프링 DI 컨테이너와 연계`
  - DI 컨테이너 상에서 동작하는 프레임워크이며 DI & AOP 기술을 그대로 활용

```java
@Controller // DI 컨테이너와의 연게
public class WelComeController { // POJO (=프레임워크에서제공하는 인터페이스 구현은 불필요)

  @Autowired // DI 컨테이너와의 연게
  MyService myService;

  @RequestMapping("/") // 애너테이션에서 정의정보를 지정
  public String home(Model model)  { // 유연한 인수 정의
    Data now = myService.getCurrentData();
    model.addAttribute("now", now); // 서블릿 API에 의존하지 않는 구현
    return "home"; // 부 구현기술에 의존하지 않는 뷰 이름 지정
  }
}
```

#### #MVC

- `풍부한 확장 포인트 제공`
  - 컨트롤러나 뷰와 같이 역할별로 필요한 인터페이스를 제공하여, 기본 동작 확장에 용이 ( 확장성높음 )
- `엔터프라이즈 애플리케이션에 필요한 기능 제공`
  - 메세지 관리, 세션 관리, 국제화, 파일 업로드 등등 방대한 기능 제공
- `서드파티 라이브러리와의 연계 지원`



### #사용

#### #Dependency

> pom.xml

```xml 
<dependency>
	<groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
</dependency>
<dependency>
	<groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
</dependency>
<dependency>
	<groupId>org.slf4j</groupId>
    <artifactId>jcl-over-slf4j</artifactId>
</dependency>
<dependency>
	<groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
</dependency>

```

#### #ContextLoaderListener

- 웹 애플리케이션에서 사용할 애플리케이션 컨텍스트를 만드려면 서블릿 컨테이너에 `ContextLoaderListener`클래스를 등록해야 함

> AppConfig.class

```java
@Configuration
public class AppConfig {
    
}
```

> web.xml

- 서블릿 컨테이너의 리스너 클래스로 `ContextLoaderListener` 정의
- 서블릿 컨테이너의 `contextClass` 파라미터에 `AnnotationConfigWebApplicationContext` 지정
- 서블릿 컨테이너 'contextConfigLocation' 파라미터에 `AppConfig` 등록

```xml
<listener>
	<listener-class>
    	org.springframework.web.context.ContextLoaderListener
    </listener-class>
</listener>
<context-param>
	<param-name>contextClass</param-name>
    <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
</context-param>
<context-param>
	<param-name>contextConfigLocation</param-name>
    <param-value>example.config.AppConfig</param-value>
</context-param>
```

#### #DispatcherServlet

- 스프링 MVC의 프론트 컨트롤러를 이용하기 위해 `DispatcherServlet` 클래스를 서블릿 컨테이너에 등록

> WebMvcConfig.class

```java
@Configuration // `DispatcherServlet`용 설정클래스
@EnableWebMvc
@ComponentScan("example.app")
public class WebMvcConfig extents WebMvcConfigurerAdapter {
    
}
```

> web.xml

```xml 
<servlet>
	<servlet-name>app</servlet-name>
    <servlet-class>org.framework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
    	<param-name></param-name>
        <param-value></param-value>
    </init-param>
        <init-param>
    	<param-name></param-name>
        <param-value></param-value>
    </init-param>
</servlet>
<servlet-mapping>
	<servlet-name>app</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```




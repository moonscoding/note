\#spring #message

---

## Message

- 문자열 관리 방식
- 다국어 지원 및 국제화 기능을 위해 사용

### #MessageSource

- MessageSource 메세지 정보의 출처를 추상화
- `getMessage()` 
  - 메세지 코드에 맞는 메시지 반환
  - 메세지 문구에 동적인 부분이 있다면 인수로 받아 처리
  - 요청한 코드에 대응하는 메시지가 없으면 `Default `반환

```java
public interface MessageSource {
  String getMessage(String code, Object[] args, Locale locale) throws NoSuchMessageException;
  String getMessage(String code, Object[] args, String defaultMessage, Locale locale);
  String getMessage(MessageSourceResolvable resolvable, Locale locale) throws NoSuchMessageException;
}
```



#### #구현클래스

- `ResourceBundleMessageSource`
  - JavaSE표준의 java.util.ResourceBundle
  - 프로퍼티 파일에서 메세지를 가져옴
- `ReloadableResourceBundleMessageSource`
  - 스프링이 제공하는 패키지 사용
  - 프로퍼티 파일에서 메시지 가져옴
  - java.util.ResourceBundle 기능확장



#### #설정

> ResourceBundleMessageSource

```java
@Bean
public MessageSource messageSource() {
  ResourceBundleMessageSource messageSource = new ReousrceBundleMessageSource();
  // 클래스패스 상에 있는 프로퍼티 파일의 이름을 확장자를 제외하고 지정
  messageSource.setBasenames("messages");
  return messageSource;
}
```

> 메세지

- messages.properties

```bash
#welcome.message={0}님, 환영합니다!
welcome.message={0}님, 환영합니다! #유니코드처리
```

- application-messages.properties

```bash
#result.succeed={0}처리가 성공했습니다.
result.succeed={0}처리가 성공했습니다. #유니코드처리
```



#### #활용

```java
@Autowired
MessageSource messageSource;

public void printWelcomeMessage() {
  String message = messageSource.getMessage(
      "result.succeed", 
      new String[] {"사용자 등록"},
      Locale.KOREAN);
  System.out.println(message); // 사용자 등록처리가 성공했습니다.
}
```





### #MessageSourceResolvable

- 메세지정보를 가져오는데 필요한 정보(code, args, defaultMessage)의 덩어리 인터페이스

```java
public interface MessageSourceResolvable {
  String[] getCodes();
  Object[] getArguments();
  String getDefaultMessage();
}
```



#### #활용

> DefaultMessageSourceResolvable 

- `messageSource`의 인수값을 프로퍼티 파일에서 처리

```java
MessageSourceResolvable functionName = new DefaultMessageSourceResolvable("functionName.userRegistration");

String message = messageSource.getMessage(
    "result.succeed", 
    new MessageSourceResolvable[]{functionName},
  	Locale.KOREAN);
```

```bash
#functionName.userRegistration=사용자등록
functionName.userRegistration=사용자등록 #유니코드처리
```



### #인코딩

- `ResourceBundleMessageSource`
  - `setDefaultEncoding()`
    - 프로퍼티 파일을 다른 인코딩 방식으로 사용할 수 있는 기능 제공

```java
@Bean
public MessageSource messageSource() {
  ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
  messageSource.setBasenames("messages");
  messageSource.setDefaultEncoding("UTF-8");
  return messageSource;
}
```

```bash
#functionName.userRegistration=사용자등록
functionName.userRegistration=사용자등록
```



### #다국어

- `MessageSource` 구현클래스는 국가별로 메시지 언어를 다르게 적용할 수 있음
- 국가코드와 조합해서 메시지를 세분화 처리 가능
  - `message_en_US.properties` & `messages_en_GB.properties`

> messages.properties

```bash
welcome.message={0}님, 환영합니다.
```

> messages_en.properties

```bash
welcome.message=Welcome, {0}!
```




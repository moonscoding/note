



# Enum

## Java Tutorial



### Enum 상속

> 일단 Enum 객체는 상속이 되지 않습니다.

- Enum이 다른 Enum을 확장할 수 없으며 상속을 통해 기존 열거형에 값을 추가할 순 없음



>  따라서 `Enum`은 인터페이스를 사용하여 상속을 구현해야 합니다.

- 열거형이 마커 인터페이스를 구현하도록 하는것 ( 즉 메소드의 선업이 없음 )
- 클라이언트는 동일한 인터페이스를 구현하는 자체 열거형을 만들 수 있음
- 그런 다음 열거형 값은 공통 인터페이스로 참조
- 요구 사항을 강화하기 위해 인터페이스가 관련 메소드를 선언하도록 할 수 있음



> [CASE1] Interface를 활용한 Enum 구현 예제

```java
public class Main {

    public static void main(String[] args) {
        List<HTTPMethodConvertible> blah = new ArrayList<>();
        blah.add(LibraryEnum.FIRST);
        blah.add(ClientEnum.BLABLABLA);
        for (HTTPMethodConvertible element: blah) {
            System.out.println(element.getHTTPMethodType());
        }
    }

    // Enum의 인터페이스
    static interface HTTPMethodConvertible {
        public String getHTTPMethodType();
    }
    
    // 인터페이스를 상속한 Enum
    static enum LibraryEnum implements HTTPMethodConvertible {
        FIRST("first"),
        SECOND("second"),
        THIRD("third");
        
        String httpMethodType;
        
        LibraryEnum(String s) {
            httpMethodType = s;
        }
        
        public String getHTTPMethodType() {
            return httpMethodType;
        }
    }
    
    // 인터페이스를 상속한 Enum
    static enum ClientEnum implements HTTPMethodConvertible {
        FOO("GET"),BAR("PUT"),
        BLAH("OPTIONS"),
        MEH("DELETE"),
        BLABLABLA("POST");
        
        String httpMethodType;
        
        ClientEnum(String s){
            httpMethodType = s;
        }
        
        public String getHTTPMethodType() {
            return httpMethodType;
        }
    }
}
```





> [CASE2] Static Method를 활용

```java
public class CodeTest {
    
    // Enum 비즈니스에 활용할 Static Method
    private static <E extends Enum<E> & Codable> E from(E[] values, String code) {
        for (E e : values)
            if (e.getCode().equals(code))
                return e;

        throw new IllegalArgumentException("Boring: " + code);
    }
    
   
    private interface Codable {
        String getCode();
        default Codable makeCode(String code) {
            
        };
    }
    
    enum Gender implements Codable {
        MALE("1"), 
        FEMALE("2");
        
        private final String code;
        
        Gender(String code) { 
            this.code = code; 
        }
        
        public String getCode() { 
            return this.code; 
        }
        
        // 비즈니스에 활용할 Static 메소드는 Enum 객체 내부에서 호출
        static Gender makeCode(String genderCode) {
            return from(values(), genderCode);
        }
    }
    
    enum FixedCode implements Codable {
        THIS("001"), THAT("002");
        
        private final String code;
        
        FixedCode(String code) { 
            this.code = code; 
        }
        
        public String getCode() { 
            return this.code; 
        }
        
        // 비즈니스에 활용할 Static 메소드는 Enum 객체 내부에서 호출
        static FixedCode makeCode(String fixedCode) {
            return from(values(), fixedCode);
        }
    }
}

출처: https://homo-ware.tistory.com/69 [人-ware : Forwards Veritas]
```


# Class Literal 

- `Class Literal` 은 `String.class , Integer.class` 등을 말함
  - `String.class`의 타입은 `Class<String>`
  - `Integer.class`의 타입은 `Class<Integer>`





### Type Token 

> Class Literal Parameter

- 타입토큰(Type Token)은 쉽게 말해 타입을 나타내는 토큰 클래스 리터럴이 타입 토큰으로 사용
  - `myMethod(Class<?> clazz)`와 같은 메서드는 타입 토큰을 인자로 받는 메서드
  - `myMethod(String.class)`로 호출하면 `String.class` 라는 클래스 리터럴을 타입 토큰 파라미터로 전달



### Heterogeneous Map

- Generic이 나온 이후 특정 타입을 가지는 Map은 `Map<String, String>` 같은 형식으로 키와 밸류의 타입으로 타입 안전성을 확보
- 정해진 특정 타입이 아니라 다양한 타입을 지원해야 하는 Heterogeneous Map이 필요하다면 타입 안전성을 확보하기 위해 다른 방식 필요



> SimpleTypeSafeMap

- `List<String>.class`와 같은 형식의 타입 토큰을 사용할 수 없음

```java
public class TypeTokenMain {
    
 public static void main(String[] args) {

        class SimpleTypeSafeMap {
        
            private Map<Class<?>, Object> map = new HashMap<>();
        
            public <T> void put(Class<T> k, T v) {
                map.put(k, v);
            }
        
            public <T> T get(Class<T> k) {
                return k.cast(map.get(k));
            }
        }

        SimpleTypeSafeMap simpleTypeSafeMap = new SimpleTypeSafeMap();

        simpleTypeSafeMap.put(String.class, "abcde");
        simpleTypeSafeMap.put(Integer.class, 123);

        // 타입 토큰을 이용해서 별도의 캐스팅 없이도 타입 안전성이 확보된다.
        String v1 = simpleTypeSafeMap.get(String.class);
        Integer v2 = simpleTypeSafeMap.get(Integer.class);

        System.out.println(v1);
        System.out.println(v2);
    }
}
```



### Super Type Token

- 슈퍼타입토큰은 앞에서 List<String>.class 라는 클래스 리터럴이 존재할 수 없다는 한계를 뛰어넘게 해주는 묘수



> **Class.getGenericsSuperclass()**
>
> - 바로 위의 수퍼 클래스 타입을 반환
> - 바로 위의 수퍼 클래스가 ParameterizedType이면, 실제 타입 파라미터들을 반영한 타입을 반환
>
> https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getGenericSuperclass--
>
> 
>
> - 어떤 객체 sub의 바로 위의 수퍼 클래스가 List<String>이라는 파라미터를 사용하고 있는 ParameterizedType이면
> - sub.getClass().getGenericsSuperClass()는 List<String> 정보가 포함되어 있는 타입을 반환



```java
public class TypeTokenMain {

    public static void main(String[] args) {

        // SimpleTypeSafeMap 부분 생략...

        class Super<T> {}

        // 수퍼 클래스에 사용되는 파라미터 타입을 이용한다. 그래서 수퍼 타입 토큰.
        class Sub extends Super<List<String>> {}

        Sub sub = new Sub();

        // 파라미터 타입 정보가 포함된 수퍼 클래스의 타입 정보를 구한다.
        Type typeOfGenericSuperclass = sub.getClass().getGenericSuperclass();

        // ~~~$1Super<java.util.List<java.lang.String>> 라고 나온다!!
        System.out.println(typeOfGenericSuperclass);
    }
}
```



> **ParameterizedType.getActualTypeArguments()**
>
> - `getGenericSuperClass() `의 설명을 보면 수퍼클래스가 `ParameterizedType`이면 타입 파라미터를 포함한 정보를 반환
> - `ParameterizedType?`
>   - 파라미터화된 형을 표현 



```java
public class TypeTokenMain {

    public static void main(String[] args) {

        // SimpleTypeSafeMap 부분 생략...


        class Super<T> {}

        class Sub extends Super<List<String>> {}

        Sub sub = new Sub();

        Type typeOfGenericSuperclass = sub.getClass().getGenericSuperclass();

        // ~~~$1Super<java.util.List<java.lang.String>> 라고 나온다!!
        System.out.println(typeOfGenericSuperclass);

        // 수퍼 클래스가 ParameterizedType 이므로 ParameterizedType으로 캐스팅 가능
        // ParameterizedType의 getActualTypeArguments()으로 실제 타입 파라미터의 정보를 구한다!!
        Type actualType = ((ParameterizedType) typeOfGenericSuperclass).getActualTypeArguments()[0];

        // 심봤다! java.util.List<java.lang.String>가 나온다!!
        System.out.println(actualType);
    }
}
```



[Class Literal 설명글](https://homoefficio.github.io/2016/11/30/%ED%81%B4%EB%9E%98%EC%8A%A4-%EB%A6%AC%ED%84%B0%EB%9F%B4-%ED%83%80%EC%9E%85-%ED%86%A0%ED%81%B0-%EC%88%98%ED%8D%BC-%ED%83%80%EC%9E%85-%ED%86%A0%ED%81%B0/)

[Reflection 설명글](https://umanking.github.io/2019/08/24/java-reflection/)
# Annotation



## Annotation ?

- 자바 소스코드에 추가적인 정보를 제공하는 메타데이터 
- 어노테이션은 클래스 / 메서드 / 변수 / 인자에 추가 가능
- 메타 데이터이기 때문에 비즈니스 로직에 직접적인 영향을 주진 않으나 이 메타데이터 정보에 따라 실행 흐름을 변경할 수 있는 코딩이 가능함
  - 1 컴파일러에게 코드 문법 에러를 체크하도록 정보를 제공 (@Override)
  - 2 소프트웨어 개발 툴이 빌드나 배치 시에 코드를 자동으로 생성할 수 있도록 정보를 제공
  - 3 실행 시 (런타임 시) 특정 기능을 실행하도록 정보를 제공



## Type ?

- `마커 어노테이션 ( Maker Annotation )`
  - @NewAnnotation
- `싱글값 어노테이션 (Single Value Annotation)`
  - @NewAnnotation(id=10)
- `멀티값 어노테이션(Multi Value Annotation)`
  - @NewAnnotation(id=10, name="hello", roles={"admin", "user"})



## Custom

### @Target

- 어노테이션이 적용될 수 있는 위치를 결정
- ElementType - Enum
  - `TYPE `
    - class, interface, enum
  - `FIELD `
    - 클래스 필드 변수
  - `METHOD`
    - 메서드
  - `PARAMETER`
    - 메서드 인자
  - `CONSTRUCTOR`
    - 생성자
  - `LOCAL_VARIABLE` 
    -  로컬 변수
  - `ANNOTATION_TYPE `
    - 어노테이션
  - `PACKAGE `
    - 패키지
  - `TYPE_PARAMETER `
    - 제네릭 타입 변수 ( 자바 8 부터)
  - `TYPE_USE`
    - 어떤 타입에도 적용 (자바 8 부터)
  - `MODULE`
    - 모듈 ( 자바 9 부터 )



### @Retention

- 어노테이션이 어느레벨까지 유지되는지 결정
- Retention - Enum
  - `SOURCE`
    - 자바 컴파일에 의해 어노테이션 삭제
  - `CLASS`
    - 어노테이션은 .class 파일에 남지만, runtime에 제공되지 않는 어노테이션 (default)
  - `RUNTIME` 
    - 런타임에도 어노테이션이 제공되어 자바 reflection으로 선언한 어노테이션에 접근 가능



### @Inherited

- 자식 클래스가 어노테이션 상속



### @Documented

- 새로 생성한 어노테이션이 자바 문서 생성시 자바 문서에도 포함시킴



### @Repeatable

- 반복 선언 (자바 8 부터)



### @Component

- 



## Reflection API

- `isAnnotationPresent ( Class<? extends Annotation> annotationClass ) -> boolean`
  - 지정한 어노테이션이 적용되었는지 여부 
  - Class에서 호출했을 때 상위 클래스에 적용된 경우에도 true 
- `getAnnotation ( Class<T> annotationClass ) -> Annotation`
  - 지정한 어노테이션이 적용되어 있으면 어노테이션을 리턴 그렇지 않다면 null
  - Class 에서 호출했을 때 상위 클래스에 적용된 경우에도 어노테이션 리턴
- `getAnnotations() -> Annotation[]`
  - 적용된 모든 어노테이션을 리턴
  - Class 에서 호출시에 상위 클래스에 적용된 어노테이션도 모두 포함
  - 적용된 어노테이션이 없으면 길이가 0인 배열 리턴
- `getDeclaredAnnotations() -> Annotation[]`
  - 직접 적용된 모든 어노테이션 리턴
  - Class에서 호출했을 때 상위 클래스에 적용된 어노테이션 포함 안함



### Field Injection

````java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface InsertIntData {
    
    int data() default 0;
    
    String value() default "-";
    
}
````

```java
public class Example { 
	@InsertIntData
    private int myAge;
}
```





### Annotation Execute

- Reflection은 성능이 좋지 않다는 치명적인 단점이 있습니다.
- 매개변수의 타입체킹도 런타임에서 진행되기 때문에 위험할 수 있습니다.



#### Field 찾기

```java
for(Field field : fieldArray) {
    String fieldName = field.getName();
    CustomAnnotation testAnnotation = field.getAnnotation(CustomAnnotation.class);
    Annotation[] annotationArray = field.getDeclaredAnnotations();
    boolean isAnnotationPresent = field.isAnnotationPresent(CustomAnnotation.class); 
    
    if(testAnnotation != null) {
        field.setAccessible(true);
        if(obj != null) {
            String fieldValue = (String) field.get(obj);
            field.set(obj, "33");
            fieldValue = (String) field.get(Obj);
        }
    }
}
```



#### Method 찾기

```java
//자바 리플렉션 getMethod로 메서드 doThis를 얻어온다
Method method = TheClass.class.getMethod("doThis”);

//메서드에 선언된 어노테이션 객체를 얻어온다
Annotation[] annotations = method.getDeclaredAnnotations(); 

for (Annotation annotation : annotations) {
    if (annotation instanceof MyAnnotation) {
        MyAnnotation myAnnotation = (MyAnnotation) annotation;

        //어노테이션에 지정한 값을 프린트한다
        System.out.println("name: " + myAnnotation.name());
    }
}
                                         
=====================================================================================
                                         
String methodName = method.getName();
CustomAnnotation testAnnotation = method.getAnnotation(CustomAnnotation.class);
Annotation[] annotationArray = method.getDeclaredAnnotations();
boolean isAnnotationPresent = method.isAnnotationPresent(CustomAnnotation.class);

if (testAnnotation != null) {

    if (obj != null) {
        method.invoke(obj);
    }
}
```



- 어노테이션 수행 클래스

```java
public class AnnotationHandler {
    
    private <T> T checkAnnotation(T targetObj, Class annotationObj) {
        Field[] fields = targetObj.getClass().getDeclaredFields();
        for (Field f : fields) {
            if(annotationObj == InsertIntData.class) {
                return checkAnnotation4InsertInt(targetObj, f);
            }
            else if(annotationObj == InsertStringData.class) {
                return checkAnnotation4InsertString(targetObj, f);
            }
        }
        return targetObj;
    }

    private <T> T checkAnnotation4InsertInt(T targetObj, Field field) {
        InsertIntData annotation = field.getAnnotation(InsertIntData.class);
        if(annotation != null && field.getType() == int.class) {
            field.setAccessible(true);
            try {  field.set(targetObj, annotation.data()); }
            catch (IllegalAccessException e) { System.out.println(e.getMessage()); }
        }
        return targetObj;
    }
    
    private <T> T checkAnnotation4InsertString(T targetObj, Field field) {
        InsertStringData annotation = field.getAnnotation(InsertStringData.class);
        if(annotation != null && field.getType() == String.class) {
            field.setAccessible(true);
            try { field.set(targetObj, annotation.data()); }
            catch (IllegalAccessException e) { System.out.println(e.getMessage()); }
        }
        return targetObj;
    }
    
    public <T> Optional<T> getInstance(Class targetClass, Class annotationClass) {
        Optional optional = Optional.empty();
        Object object;
        try {
            object = targetClass.newInstance();
            object = checkAnnotation(object, annotationClass);
            optional = Optional.of(object);
        }catch (InstantiationException | IllegalAccessException e) { System.out.println(e.getMessage()); }
        return optional;
    }
    
}
```

- getInstance
- checkAnnotation
- checkAnnotation4InsertInt & checkAnnotation4InsertString







[annotation a](https://dukwon.tistory.com/42)

[annotation_b](https://elfinlas.github.io/2017/12/14/java-custom-anotation-01/)

[annotation_c](https://advenoh.tistory.com/21)

[annotation_d](http://www.nextree.co.kr/p5864/)


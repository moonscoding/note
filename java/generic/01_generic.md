# Generic

## Class<T>

- `class`
  - `ExampleClass<T>`
    - T가 Class Literal은 아님
    - Compile 단계에서 클래스의 구성요소를 설정
- `mehtod`
  - `exampleMethod(T t)`
    - T의 instance
  - `exampleMethod(Class<T> clazzT)`
    - Class Literal으로 사용 ( 타입 명시 )
  - `exampleMethod(Class<?> classType)`
    - Class Literal으로 사용 ( 모든 타입 )


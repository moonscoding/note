\#spring #data

## 스프링데이터란

- 스프링 데이터는 DB나 캐시서버등의 데이터에 접근하기 위한 코드를 줄이는 것을 목적
- 스프링은 `DDD(Domain-Driven Design)`에서의 데이터 접근 계층을 구현하기 위한 구성 요소인 `Repository`를 구현하는 것을 권장
  - 스프링에서는 `@Repository` 가 제공



> Spring Data Commons

- Spring Data JPA
- Spring Data MongoDB
- Spring Data Redis
- Spring Data Solr
- Spring Data GemFire
- Spring Data REST



### #스프링데이터JPA

- JPA를 이용해서 데이터에 접근하기 위한 Repository를 구현하는데 걸리는 부하를 최소화합니다.
- Entity 참조하거나 갱신하는 것이 가능
- JPA 이벤트 리스너를 활용해서 데이터를 생성 또는 변경한 날짜/시간 등의 감시 정보를 자동으로 Entity에 바인딩할 수 있음



### #스프링데이터아키텍처

- 내부처리흐름
  - Entity 전용 Repository 작성
  - 스프링 데이터 JPA가 제공하는 `JpaRepository` 인터페이스 상속
  - `SimpleJpaRepository` 인터페이스를 상속한 `JpaRepository`를 상속했으므로 해당 API 호출가능
  - Entity 에 대한 애플리케이션에서 데이터 접근을 하는 경우, 대응하는 Repository를 통해 스프링 데이터 JPA를 각각 호출



### #JpaRepository

> Repository 인터페이스 정의

- 취급하고 싶은 Entity 타입과 그것의 기본키 타입을 제네릭으로 지정한 Repository 인터페이스 정의
- 스프링 데이터 JPA가 표준으로 제공하는 메서드만 이용하는 경우 메서드를 정의할 필요 없음

```java
public interface RoomRepository extends JpaRepository<Room, Integer> {

}
```



> JpaRepository API

- `save`
  - <S extends T> S save(S entity)
  - <S extends T> List<S> save(Interable<S> entities)
    - 지정된 Entity에 대한 저장 작업을 처리하도록 모음
    - Entity 기본키가 설정되어 있지 않다면 EntityManager 메서드 호출
    - 값이 설정돼있다면 merge() 호출
  - [실행될 가능성이 있는 EntityManager 메서드]
    - persist() & merge()
- `flush`
  - void flush()
    - EntityManager에 모아둔 Entity에 대한 저장작업실행, DB반영
  - [실행될 가능성이 있는 EntityManager 메서드]
    - flush()
- `saveAndFlush`
  - T saveAndFlush(T entity)
    - 지정된 Entity에 대한 저장작업을 처리하도록 모음
    - 그 저장작업을 실행해 DB에 저장결과반영
  - [실행될 가능성이 있는 EntityManager 메서드]
    - save() & flush()
- `delete`
  - void delete(T entity)
  - void delete(Iterable<? extends T> entities)
    - 지정된 Entity에 대한 삭제작업을 처리하도록 모음
  - [실행될 가능성이 있는 EntityManager 메서드]
    - remove() & contains() & merge()
  - void delete(ID id)
    - 기본키를 지정한 Entity에 대해 삭제작업을 처리하도록 모음
    - 삭제대상이 되는 Entity가 존재하지 않으면 EmptyResultDataAccessException이 발생
  - [실행될 가능성이 있는 EntityManager 메서드]
    - remove() & contains() & merge() & find()
- `deleteAll`
  - void deleteAll()
    - 모든 Entity에 대한 삭제작업을 처리하도록 모음
    - 삭제할 DB의 모든 레코드의 영속성 컨텍스트에 읽어 들여야 하기 때문에 만약 삭제 건수가 많다면 deleteAllInBatch() 사용
  - [실행될 가능성이 있는 EntityManager 메서드]
    - delete(Iterable<? extends T> entities)
- `deleteInBatch`
  - void deleteInBatch(Interable entities)
  - void deleteAllInBatch()
    - 모든 Entity에 대한 삭제작업을 영속성 컨텍스트로 읽어들이지 않고 DB에서 삭제
    - 호출전 EntityManager에서 관리되고 있는 Entity는 DB와 동기화되지 않은 상태
    - 삭제후 에도 findOne()으로 검색될 수 있음
  - [실행될 가능성이 있는 EntityManager 메서드]
    - createQuery() & TypedQuery() & executeUpdate()
- `findOne`
  - T findOne(ID id)
    - 기본키로 Entity를 DB에서 찾아오기 위한 메서드
    - EntityManager.find() 경우와 비슷하게 이미 관리 상태인 Entity는 접근 발생하지 않음
  - [실행될 가능성이 있는 EntityManager 메서드]
    - find()
- `findAll`
  - List findAll()
  - List findAll(Sort sort)
    - List findAll(Iterable ids)모든 혹은 지정된 여러 기본키에 해당하는 Entity를 DB에서 가져옴
    - findOne과 달리 관리대상에 있는 경우에도 접근이 발생Sort 지정 경우에 정렬 순서를 지정
    - ORDER BY절 추가기본키를 지정한 경우 IN절 추가
  - [실행될 가능성이 있는 EntityManager 메서드]
    - createQuery() & TypedQuery() & getResultList()
  - Page findAll(Pageable pageable)
    - 지정한 페이지(정렬순서, 페이지번호, 페이지에 표시하 건수)에 해당하는 Entity만 가져옴
    - 주로 웹앱에서 페이지 처리에 사용
  - [실행될 가능성이 있는 EntityManager 메서드]
    - createQuery() & TypedQuery() & getResultList()
- `exists`
  - boolean exists(ID id)
    - 기본키를 지정해 Entity가 DB에 존재하는지 확인
    - 대상 Entity가 관리 상태에 있는 경우에도 접근 발생
  - [실행될 가능성이 있는 EntityManager 메서드]
    - createQuery() & TypedQuery() & getSingleResult()
- `count`
  - long count()
    - DB 상의 모든 Entity 건수를 취득
    - 반드시 데이터베이스 접근 발생
  - [실행될 가능성이 있는 EntityManager 메서드]
    - createQuery() & TypedQuery() & getSingleResult()



### #예외변환

- `@Repository`에서 발생한 예외를 비검사예외인 `DataAccessException`으로 변환하는 구조제공

- Repository가 프락시 클래스로 생성되므로 프락시 클래스에 대해 같은 예외변환 구조가 적용

- 따라서 JPA 구현에서 발생한 예외는 비검사예외에 래핑되어 호출된 곳으로 던져짐

- JDBC에서의 데이터접근과 동일하게 예외처리구현

  - `@EnableJpaRepository`
  - `<jpa:repositories>`

  

> DataAccessException 예외

- `DuplicationKeyException`
  - 데이터를 변경시에 무결성 제약위반이 발생한 경우
- `PessimiticLockingFailureException`
  - JPA의 비관적잠금에 실패하고 JPA구현이 `PessimiticLockingException`을 발생시킨 경우
- `OptimisticLockingFailureException`
  - JPA의 낙관적잠금에 실패하고 JPA구현이 `OptimisticLockingException`을 발생시킨 경우



> SQL을 강제로 실행하고 예외를 처리

```java
try {
  RoomRepository.saveAndFlush(room);
} catch(OptimisticLockingFailureException e) {
 
}
```


\#spring #JDBC

## JDBC

- SQL의 실제내용과는 상관없으나 반복적으로 또는 공통적으로 수행되는 대행



> 공통적 or 반복적으로 수행되는 기능

- 커넥션의 연결과 종료
- SQL문의 실행
- SQL문의 샐행과 행에 대한 반복처리
- 예외처리

> 개발자가 처리해야하는 기능

- SQL문 정의
- 파라미터 설정
- 결과(ResultSet)에 대한 비즈니스로직



### #JdbcTemplate

> JdbcTemplate 설정과 활용

```java
@Configuration
public class AppConfig {
  @Bean
  public JdbcTemplate jdbcTemplate (Database database) {
    return new JdbcTemplate(database);
  }
}
```

```java
@Autowired
JdbcTemplate jdbcTemplate;

public String findUserName(String userId) {
  String sql = "SELECT user_name FROM user WHERE user_id = ?";
  return jdbcTemplate.queryForObject(sql, String.class, userId);
}
```



### #API

- `queryForObject`
  - 하나의 결과 레코드 중 하나의 칼럼 값을 가져옴
  - `RomMapper`와 함께 사용하여 레코드 정보를 객체에 매핑
- `queryForMap`
  - 하나 결과 레코드 정보를 Map 형태로 매핑
- `queryForList`
  - 여러개의 결과 레코드를 다룸
  - List의 한 요소가 한 레코드
  - 한 레코드의 정보는 `queryForObject`, `queryForMap`과 같음
- `query`
  - `ResultSetExtractor.RowCallbackHandler`과 함께 조회시 사용
- `update`
  - 데이터를 변경하는 SQL (insert, update, delete)



### #DAO

> DAO(Data Access Object) 구현

```java
@Component
public class JdbcRoomDao {
    
  @Autowired
  JdbcTemplate jdbcTemplate;

  public int findMaxCapacity() {
    String sql = "SELECT MAX(capacity) FROM room";
    return jdbcTemplate.queryForObject(sql, Integer.class);
  }
}
```

> DAO 실행

```java
public static void main(String[] args) {
  ApplicationContext context = new ClassPathXmlApplicationContext("JdbcTemplateConfig.xml");
  jdbcRoomDao dao = context.getBean("jdbcRoomDao", JdbcRoomDao.class);
  int maxCapacity = dao.findMaxCapacity();
  System.out.println(maxCapacity);
}
```



### #NamedParameterJdbcTemplate

#### #Map

- 메서드에 바인드변수의 이름을 키로 사용하는 Map 활용

```java
@Component
public class JdbcRoomNameDao {
  @Autowired
  NamedParameterJdbcTemplate namedParameterJdbcTemplate;

  public String findRoomNamedById(String roomId) {
    String sql = "SELECT room_name FROM room WHERE room_id = :roomId"; // :바인드변수명
    Map<String, Object> params  = new HashMap<String, Object>();
    params.put("roomId", roomId);
    return namedParameterJdbcTemplate.queryForObject(sql, params, String.class);
  }
}
```

#### #SqlParameterSource

- `SqlParameterSource`인터페이스의 구현클래스를 인수로 받음
  - `MapSqlParameterSource`
  - `BeanPropertySqlParameterSource`
- `Map`사용에 비해 설정이 편리

> MapSqlParameterSource

- `addValue()`로 파라미터값 설정

```java
MapSqlParameterSource map = new MapSqlParameterSource()
  .addValue("roomId", "A001")
  .addValue("roomName", "회의실")
  .addValue("capacity", 10);
```

> BeanPropertySqlParameterSource

- SQL문의 파라미터 값을 설정할때, 빈객체를 사용

```java
Room room = new Room("A001", "회의실", 10);
BeanPropertySqlParameterSource map = new BeanPropertySqlParameterSource(room);
```



### #CRUD

> 1건조회

```java
public Room getRoomById(String roomId) {
  String sql = "SELECT room_id, room_name, capacity FROM room WHERE room_id = ?";
  Map<String, Object> result = jdbcTemplate.queryForMap(sql, roomId);
  Room room = new Room();
  room.setRoomId((String) result.get("room_id"));
  room.setRoomName((String) result.get("room_name"));
  room.setRoomCapacity((Integer) result.get("room_capacity"));
  return room;
}
```

> 여러건조회

```java
public List<Room> getAllRoom() {
    String sql = "SELECT room_id, room_name, capacity FROM room";
    List<Map<String, Object>> resultList = jdbcTemplate.queryForList(sql);
    List<Room> roomList = new ArrayList<Room>();
    for(Man<String, Object> result : resultList) {
        Room room = new Room();
        room.setRoomId((String) result.get("room_id"));
        room.setRoomName((String) result.get("room_name"));
        room.setRoomCapacity((Integer) result.get("room_capacity"));
        roomList.add(room);
    }
    return roomList;
}
```

> 결과 0

- `queryForList` 에서는 빈 List를 반환
- 그 밖의 경우는 `EmptyResultDataAccessException`예외발생

> 테이블내용변경

- 등록, 수정, 삭제 상관없이 `update`사용

- 조회된 데이터가 없을 경우 `EmptyResultDataAccessException` 예외발생
- 반환값으로 성공여부 파악

```java
@Autowired
JdbcTemplate jdbcTemplate;

public int insertRoom(Room room) {
  String sql = "INSERT INTO room(room_id, room_name, capacity) VALUES(?, ?, ?)";
  return jdbcTemplate.update(sql, room.getRoomById(), room.getRoomName(), room.getCapacity());
}

public int updateRoomById(Room room) {
  String sql = "UPDATE room SET room_name=?, capacity=? WHERE room_id=?";
  return jdbcTemplate.update(sql, room.getRoomName(), room.getCapacity(), room.getRoomById());
}

public int deleteRoomById(String roomId) {
  String sql = "DELETE FROM room WHERE room_id=?";
  return jdbcTemplate.update(sql, roomId);
}
```



### #POJO Mapper

- JDBC는 결과값을 자바의 `데이터타입`이나, `Map`, `List`형태로 반환
- `POJO(자바 Object)`를 통해 결과값을 변환하는 방법



#### #RowMapper

- JDBC의 ResultSet을 순차적으로 읽으며 POJO 매핑

- ResultSet은 하나의 행으 하나의 POJO로 변환

> 구현클래스

- mapRow 메서드를 만들어 그 안에 `ResultSet`과 `POJO` 간의 변환을 처리
- 메소드의 반환값을 제네릭으로 지정한 파라미터로 처리
- RowMapper 구현클래스는 ResultSet 커서를 제어할 수 없음 (ResultSetExtrator와 차이)

```java
public class RoomRowMapper implements RowMapper<Room> {
    @Override
    public Room mapRow(ResultSet resultSet, int rowNum) throws SQLException {
      Room room = new Room();
      room.setRoomId(resultSet.getString("room_id"));
      room.setRoomName(resultSet.getString("room_name"));
      room.setCapacity(resultSet.getInt("capacity"));
      return room;
    }
}
```

> DAO

```java
public Room getRoomById(String roomId) {
  String sql = "SELECT room_id, room_name, capacity FROM room WHERE room_id = ?";
  RoomRowMapper rowMapper = new RoomRowMapper();
  return jdbcTemplate.queryForObject(sql, rowMapper, roomId);
}

public List<Room> getAllRoom() {
  String sql = "SELECT room_id, room_name, capacity FROM room";
  RoomRowMapper rowMapper = new RoomRowMapper();
  return jdbcTemplate.query(sql, rowMapper);
}
```

> 람다 (익명구현)

```java
public List<Room> getAllRoom() {
   String sql = "SELECT room_id, room_name, capacity FROM room";
   return jdbcTemplate.query(sql, (resultSet, rowNum) -> {
     Room room = new Room();
     room.setRoomId(resultSet.getString("room_id"));
     room.setRoomName(resultSet.getString("room_name"));
     room.setCapacity(resultSet.getInt("capacity"));
     return room;
   });
}
```



##### #BeanPropertyRowMapper

- 미리 약속된 매핑규칙에 따라 질의의 결과정보를 `POJO`로 자동매핑

- 자동매핑을 위해 자바 `리플렉션` 사용 (성능저하를 야기 - RowMapper가 성능엔 우수)

> 매핑규칙과 제약사항

- 매핑규칙
  - ResultSet의 칼럼이름과 매핑대상인 타켓클래스의 프로퍼티 이름매핑
  - 칼럼이름을 언더스코어 문자로 구분
  - 카멜케이스 표기법으로 조합한 것과 타켓클래스 프로퍼티 이름이 일치하면 칼럼값 매핑
  - 기본 데이터 모두 지원
- 제약사항
  - 매핑될 타켓클래스는 다른 클래스에 포함되지 않은 최상위 클래스
  - 매핑될 타켓클래스는 기본 생성자나 인수가 없는 생성자가 있어야함

> DAO

```java
public Room getRoomBeanPropertyById(String roomId) {
  String sql = "SELECT room_id, room_name, capacity FROM room WHERE room_id=?";
  RowMapper<Room> rowMapper = new BeanPropertyRowMapper<Room>(Room.class);
  return jdbcTemplate.queryForObject(sql, rowMapper, roomId);
}
```



#### #ResultSetExtrator

- JDBC의 ResultSet을 자유롭게 제어하며 원하는 POJO 형태로 매핑
- ResultSet의 여러행 사이를 이동가능 (RowMapper와 차이)

> 구현클래스

- extraData 메서드를 만들어 그 안에서 `ResultSet`과 `POJO` 변환처리
- 반환값이 지정한 타입파라미터값과 같아야함
- 중복키가 있을 수 있어 `Map`을 사용

```java
public class RoomListResultSetExtractor implements ResultSetExtractor<List<Room>> {

  @Override
  public List<Room> extractData(ResultSet rs) throws SQLException, DataAccessException {
    Map<String,Room> map = new LinkedHashMap<String,Room>();
    Room room = null;
    while(rs.next()) {
      String roomId = rs.getString("room_id");
      room = map.get(roomId);
      if(room == null) {
        room = new Room();
        room.getRoomId(roomId);
        map.put(roomId, room); // map insert
      }
      String equipmentId = rs.getString("equipment_id");
      if(equipmentId != null) {
        Equiment equipment = new Equiment();
        room.getEquipmentList().add(equipment); // setter
      }
    }
    if(map.size() == 0) {
      throw new EmptyResultDataAccessException(1);
    }
    return new ArrayList<Room>(map.values());
  }
}
```

> DAO

```java
public List<Room> getAllRoomWithEquipment() {
  String sql = "SELECT r.room_id, r.room_name, r.capacity, " +
    "e.equipment_id, e.equipment_name, e.equipment_count, e.equipment_remarks " +
    "FROM room r LEFT JOIN equipment e " +
    "ON r.room_id = e.room_id"

  RoomListResultSetExtractor extracter = new RoomListResultSetExtractor();
  return jdbcTemplate(sql, extractor);
}

public Room getRoomWithEquipmentById(String roomId) {
  String sql = "SELECT r.room_id, r.room_name, r.capacity, " +
    "e.equipment_id, e.equipment_name, e.equipment_count, e.equipment_remarks " +
    "FROM room r LEFT JOIN equipment e " +
    "ON r.room_id = e.room_id " +
    "WHERE room_id = ?";

  RoomListResultSetExtractor extractor = new RoomListResultSetExtractor();
  List<Room> rooomList = jdbcTemplate.query(sql, extractor, roomId);
  return roomList.get(0);
}
```



#### #RowCallbackHandler

- 반환값 없음
- ResultSet에서 읽은 데이터를 파일형태로 출력 혹은 조회된 데이터를 검증하는 용도

> 구현클래스

```java
public class RoomRowCallbackHandler implements RowCallbackHandler {
    @Override
    public void processRow(ResultSet rs) throws SQLException {
        try {
            BufferedWriter writer = new BufferedWriter(
                new OutputStreamWriter(new FileOutputStream(File.createTempFile("room_", ".csv")), "UTF-8")
            );
            while(rs.next()) {
                Object[] array = new Object[]{
                    rs.getString("room_id"),
                    rs.getString("room_name"),
                    rs.getInt("capacity")
                };
                String reportRow = StringUtils.arrayToCommaDelimitedString(array);
                writer.write(repoertRow);
                writer.newLine();
            }
        } catch(IOException e) {
            throw new SQLException(e);
        } finally {
            writer.close();
        }
    }
}
```

> DAO

- csv 파일을 생성하는 메소드정의

```java
public void reportRow() {
  String sql = "SELECT room_id, room_name, capacity FROM room";
  RoomRowCallbackHandler handler = new RoomRowCallbackHandler();
  jdbcTemplate.queryt(sql, handler);
}
```



### #일괄처리

#### #배치처리

- 대용량데이터를 처리할때 SQL문을 각각 실행하는 것이 아닌 배치형태로 모아서 실행
- `batchUpdate()` 메소드활용 ( JdbcTemplate & NamedParameterJdbcTemplate )

#### #저장프로시저호출

- 저장프로시저는 JdbcTemplate의 `call()` 메소드
- 저장함수는 JdbcTemplate의 `execute()` 메소드
- `Proceduce` 클래스나 `SimpleJdbcCall`클래스를 사용할 수 있음
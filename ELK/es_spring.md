# Elasticsearch + Spring

es를 spring에 연동하는 방법을 알아보겠습니다.



## Dependency

> pom.xml

- ES에서 제공하는 REST API에는 공식적으로 두가지 API가 존재
  - `java Low Level REST Client`
  - `java High Level REST Client`
- 기본적인 PUT / POST / GET / DELETE 등등 low level 버전으로 커버가 가능
- high level 버전의 가장 큰 특징은 `비동기메소드`를 제공하는 것

```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-client</artifactId>
    <version>5.6.3</version>
</dependency>
```



## JAVA HIGH level RESTClient

<https://www.elastic.co/guide/en/elasticsearch/client/java-rest/master/java-rest-high.html>

<https://snapshots.elastic.co/javadoc/org/elasticsearch/client/elasticsearch-rest-high-level-client/8.0.0-SNAPSHOT/index.html>

- HighLevel은 LowLevel의 맨 위에서 작동
- 주요 목표는 요청 객체를 인수로 받고 응답 객체를 반환하는 API 특정 메소드를 노출하여
  요청 정렬 및 응답 등등을 클라이언트 자체에서 처리하도록 하는 것입니다.
- 각 API는 동기적으로 또는 비동기적으로 호출 가능
  - 동기 메서드는 응답개체를 반환 
  - 비동기 메서드는 응답 또는 오류가 수신되면 알림을 받는 리스너 인수가 필요 
    ( LowLevel 클라이언트에서 관리하는 스레드풀에서 )
- HighLevel은 Elasticsearch 핵심 프로젝트에 따라 다름
- TransportClient와 동일한 요청인수를 받아들이고 동일한 응답 개체를 반환





## JAVA LOW level RESTClient

- HTTP를 통해 ES 클러스터와 통신하고 marshaling 요청을 남겨 응답을 사용자에게 marshaling 하지 않게 합니다.
  - marshaling 
    - 한 객체의 메모리에서 표현방식을 저장 또는 전송에 적합한 다른 데이터 형식으로 변환하는 과정
    - 데이터를 컴퓨터 프로그램의 서로 다른 부분 간에 혹은 한 프로그램에서 다른 프로그램으로 이동해야 할 때도 사용

### 구현

> ElasticAPI

```java
@Component
public class ElasticAPI {
    
    @Value("${elasticsearch.host}")
    private String host;
    
    @Value("${elasticsearch.port}")
    private int port;
    
    /**
     * 엘라스틱서치에서 제공하는 api를 이용한 전송메소드
     * @param method
     * @param url
     * @param obj
     * @param jsonData
     * @return
     */
    public Map<String, Object> callElasticApi(String method, String url, Object obj, String jsonData) {
        Map<String, Object> result = new HashMap<>();

        String jsonString;
        //json형태의 파라미터가 아니라면 gson으로 만들어주자.
        if (jsonData == null) {
            Gson gson = new Gson();
            jsonString = gson.toJson(obj);
        } else {
            jsonString = jsonData;
        }

        //엘라스틱서치에서 제공하는 restClient를 통해 엘라스틱서치에 접속한다
        try(RestClient restClient = RestClient.builder(new HttpHost(host, port)).build()) {
            Map<String, String> params =  Collections.singletonMap("pretty", "true");
            //엘라스틱서치에서 제공하는 response 객체
            Response response = null;

            //GET, DELETE 메소드는 HttpEntity가 필요없다
            if (method.equals("GET") || method.equals("DELETE")) {
                response = restClient.performRequest(method, url, params);
            } else {
                HttpEntity entity = new NStringEntity(jsonString, ContentType.APPLICATION_JSON);
                response = restClient.performRequest(method, url, params, entity);

            }
            //앨라스틱서치에서 리턴되는 응답코드를 받는다
            int statusCode = response.getStatusLine().getStatusCode();
            //엘라스틱서치에서 리턴되는 응답메시지를 받는다
            String responseBody = EntityUtils.toString(response.getEntity());
            result.put("resultCode", statusCode);
            result.put("resultBody", responseBody);
        } catch (Exception e) {
            result.put("resultCode", -1);
            result.put("resultBody", e.toString());
        }
        return result;
    }
    
}
```

> ElasticSpringExampleApplicationTest

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class ElasticSpringExampleApplicationTests {

	@Autowired
	ElasticApi elasticApi;

	private final String ELASTIC_INDEX = "test_index";
	private final String ELASTIC_TYPE = "test_type";

	@Test
	public void 엘라스틱서치_POST_전송() {
		String url = ELASTIC_INDEX + "/" + ELASTIC_TYPE;
		Weather weather = new Weather();
		weather.setCity("Seoul");
		weather.setTemperature(10.2);
		weather.setSeason("Winter");

		Map<String, Object> result = elasticApi.callElasticApi("POST", url, weather, null);
		System.out.println(result.get("resultCode"));
		System.out.println(result.get("resultBody"));
	}


	@Test
	public void 엘라스틱서치_PUT_전송() {
		String id = "122345";
		String url = ELASTIC_INDEX + "/" + ELASTIC_TYPE+"/"+id;
		Weather weather = new Weather();
		weather.setCity("Tokyo");
		weather.setTemperature(12.3);
		weather.setSeason("Winter");

		Map<String, Object> result = elasticApi.callElasticApi("PUT", url, weather, null);
		System.out.println(result.get("resultCode"));
		System.out.println(result.get("resultBody"));
	}


	@Test
	public void 앨라스틱서치_GET_전송() {
		String id = "122345";
		String url = ELASTIC_INDEX + "/" + ELASTIC_TYPE+"/"+id;
		Map<String, Object> result = elasticApi.callElasticApi("GET", url, null, null);
		System.out.println(result.get("resultCode"));
		System.out.println(result.get("resultBody"));
	}


	@Test
	public void 앨라스틱서치_DELETE_전송() {
		String id = "122345";
		String url = ELASTIC_INDEX + "/" + ELASTIC_TYPE+"/"+id;
		Map<String, Object> result = elasticApi.callElasticApi("DELETE", url, null, null);
		System.out.println(result.get("resultCode"));
		System.out.println(result.get("resultBody"));
	}
}
```



### 인증

> callElasticApiAuth

- header 처리 - Authorization 부분은 `<username>:<password>`로 구성하고 이것을 base64로 인코딩처리
- 생선된 `Header[] headers`를 performRequest내에 파라미터를 추가

```java
public Map<String, Object> callElasticApiAuth(String method, String url, Object obj, String jsonData) {
        Map<String, Object> result = new HashMap<>();

        String jsonString;
        //json형태의 파라미터가 아니라면 gson으로 만들어주자.
        if (jsonData == null) {
            Gson gson = new Gson();
            jsonString = gson.toJson(obj);
        } else {
            jsonString = jsonData;
        }

        //엘라스틱서치에서 제공하는 restClient를 통해 엘라스틱서치에 접속한다
        try(RestClient restClient = RestClient.builder(new HttpHost(host, port)).build()) {
            String auth = user+":"+password;
            String basicAuth  = "Basic "+ Base64.getEncoder().encodeToString(auth.getBytes());
            Header[] headers = {
                    new BasicHeader("Authorization", basicAuth)
            };
            Map<String, String> params =  Collections.singletonMap("pretty", "true");
            //엘라스틱서치에서 제공하는 response 객체
            Response response = null;

            //GET, DELETE 메소드는 HttpEntity가 필요없다
            if (method.equals("GET") || method.equals("DELETE")) {
                response = restClient.performRequest(method, url, params, headers);
            } else {
                HttpEntity entity = new NStringEntity(jsonString, ContentType.APPLICATION_JSON);
                response = restClient.performRequest(method, url, params, entity, headers);

            }
            //앨라스틱서치에서 리턴되는 응답코드를 받는다
            int statusCode = response.getStatusLine().getStatusCode();
            //엘라스틱서치에서 리턴되는 응답메시지를 받는다
            String responseBody = EntityUtils.toString(response.getEntity());
            result.put("resultCode", statusCode);
            result.put("resultBody", responseBody);
        } catch (Exception e) {
            result.put("resultCode", -1);
            result.put("resultBody", e.toString());
        }
        return result;
    }
}
```









[Spring + ES]<https://yookeun.github.io/elasticsearch/2017/11/05/elastic-api/>

[SpringData + ES] <https://lng1982.tistory.com/284>

[High vs Low] <https://dzone.com/articles/java-high-level-rest-client-elasticsearch>
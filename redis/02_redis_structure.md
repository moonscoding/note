\#redis #structure

- 레디스와 데이터 구조
  - 정렬된셋(Sorted Set)
    - Set 데이터형에 저장된 요소에 정렬 기능이 추가된 구조
    - 3가지 데이터형과 문자열로 구분
  - 거대한 키-값의 저장소 / 거대한 맵( Map 데이터구조) 저장소
  - 장점 
    - 매우 단순한 구조
  - 단점
    - 저장된 데이터를 가공하는 방법에 제한



- 레디스 데이터 구조와 명령어
  - 명령어
    - 데이터 처리 명령
    - 키 관리 명령
    - 서버 관리 명령
  - 내용
    - 데이터 논리 저장구조
    - 데이터 처리 명령
    - 데이터 유형별 처리 예
    - 키 관리 명령



- 문자열 데이터
  - 저장 가능한 문자열의 크기는 512MB, 넘길시에는 Error 반환
- 문자열 데이터 입력과 조회
  - 레디스는 동시에 여러 키와 데이터를 저장하고 조회하는 mget / mset 명령을 제공
  - set / get 명령앞에 Muti의 첫 글자인 M을 붙인 것



### #get/set



### #mget/mset

- mset - O(N)
  -  항상 OK 응답
- mget - O(N)
  - [멀티벌크응답] 조회된 키 값 목록



```shell
> mset key1 value1 key2 value2 key3 value3
> mget key54 key1 key2 key3
(nil)
"value1"
"value2"
"value3"
```



### #setnx/msetnx

- msetnx - O(N)
  - n개의 키와 값의 쌍을 요청했을때 이미 존재하는 값이 하나라도 있으면, 입력된 모든 키와 값은 저장하지 않음
  - [숫자응답] 주어진 모든 키가 저장되었을 때 1, 저장되지 않았을 때 0



```shell
> mset nx:key1 value1 nx:key3 value3
> setnx nx:key3 'new value'
(integer) 0
> mget nx:key1 nx:key3
"value1"
"value3"
> msetnx nx:key2 'new value2' nx:key3 'new value3'
(integer) 0
> mget nx:key1 nx:key2 nx:key3
"value1"
(nil)
"value3"
```



### #getset

- getset - O(1)
  - 입력된 값을 저장하고 이전에 저장된 값을 돌려준다
  - [벌크응답] 이전에 저장된 값

```shell
> set mykey1 'old value'
OK
> getset mykey1 'new value'
"old value"
> getset newkey 'sample'
(nil)
> get newkey
"sample"
```



### #증가감소

- 레디스는 문자열 데이터가 숫자일 때, incr, decr 같은 숫자 증명 명령을 사용할 수 있음
- 숫자의 증감을 처리하기 위한 명령어로 incrby, decrby 명령이 있음

### #incrby/decrby

- incrby - O(1)
  - [숫자응답] 명령이 실행된 후의 키의 값
- decrby - O(1)
  - [숫자응답] 명령이 실행된 후의 키의 값

```shell
> incr test:key1
(integer) 1
> incrby test:key1 5
(integer) 6
> incrby test:key1 -1
(integer) 5
> decr test:key1
(integer) 4
> decrby test:key1 +3
(integer) 1
> incrby test:key1 5
(integer) 6
> decrby test:key1 -3
(integer) 9
```


\#MySQL #exist

> 참조페이지
>
> http://jason-heo.github.io/mysql/2014/05/28/mysql-in-vs-exists-vs-inner-join.html

## Exists vs In

- IN 연산자
  - 특정 Table(View) 데이터의 Row값에 따른 다른 Table 데이터를 추출해내고자 할 때 사용합니다.
  - 그래서 IN 연산자를 Row가 있는지 체크하는 용도로 사용하기도 합니다.
- EXISTS 연산자
  - Row가 있는지 체크하는 용도로는 EXISTS 연산자가 더 적합합니다. 
  - EXISTS 연산자는 Row가 존재하는지 확인 후 더 이상 수행되지 않습니다. 그러나 IN 연산자는 실제 존재하는 모든 값을 확인 합니다.

```sql
SELECT item
FROM TABLE_A as a
WHERE NOT EXISTS (
	SELECT item
    FROM TABLE_B as b
    WHERE a.item = b.item
)
```



## Exists vs In() vs Inner Join

- Exists & In & Inner Join은 상호 변환이 가능하며 성능의 차이가 있습니다.
  - 예제 :: n번에 속한 고객의 주문 건수를 구하여라
  - 성능 :: Inner Join > In > Exists
    - 하나 경우에 따라 Exists 연산자가 더 빠른 경우도 있습니다.

> INNER JOIN

```sql
SELECT COUNT(*)
FROM orders INNER JOIN customer
  ON orders.o_custkey = customer.c_custkey
WHERE customer.c_nationkey = 10;
```

> IN()

```sql
SELECT COUNT(*)
FROM orders
WHERE o_custkey IN (
  SELECT c_custkey FROM
  customer WHERE c_nationkey = 10
);
```

> EXISTS

```sql
SELECT COUNT(*)
FROM orders
WHERE EXISTS (
  SELECT 1 FROM customer
  WHERE c_nationkey = 10 
  AND c_custkey = orders.o_custkey
);
```

- EXISTS & INNER JOIN 
  - 



- NOT IN & NOT EXISTS & LEFT OUTER JOIN
  - 예제 :: 고객 ID가 존재하지 않는 케이스를 구하여라
  - 성능 :: Left Join > In > Exists

> LEFT JOIN

```sql
SELECT COUNT(*)
FROM orders t1 LEFT JOIN customer t2
  ON t1.o_custkey = t2.c_custkey
WHERE t2.c_custkey IS NULL;
```

> NOT IN

```sql
SELECT COUNT(*)
FROM orders
WHERE o_custkey NOT IN (
  SELECT c_custkey FROM customer
);
```

> NOT EXISTS

```sql
SELECT COUNT(*)
FROM orders
WHERE NOT EXISTS (
  SELECT 1 FROM customer
  WHERE c_custkey = orders.o_custkey
);
```




 

# MYSQL DATE 조작

> SELECT DATE 조작
>
> - DATE_FORMAT

<https://www.w3schools.com/sql/func_mysql_date_format.asp>

```sql
select date_format(D_time,"%Y-%m-%d") from 테이블명 
select date_format(D_time,"%H:%i::%s") from 테이블명
```



> 더하고빼기
>
> - DATE_ADD
> - DATE SUB

<https://www.w3schools.com/sql/func_mysql_date_add.asp>

```sql
SELECT DATE_ADD("2017-06-15", INTERVAL 10 DAY);
```


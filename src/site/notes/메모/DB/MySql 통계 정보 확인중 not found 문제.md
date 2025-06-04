---
{"dg-publish":true,"permalink":"//db/my-sql-not-found/"}
---



```sql
show tables like '%_stats'

select * from innodb_index_stats 
where database_name='test' and table_name='emp_table'

```

- 위의 쿼리 실행시  `innodb_index_stats` 테이블을 찾을 수 없다는 에러가 출력됐다.
- 결과적으로 아래의 쿼리를 수행해야한다.

```sql

use mysql;
select * from innodb_index_stats 
where database_name='test' and table_name='emp_table'


select * from mysql.innodb_index_stats 
where database_name='test' and table_name='emp_table'

```


#### MySql database
- MySQL 서버에 기본적으로 제공되는 시스템 데이터베이스
- MySQL 서버의 사용자 계정, 권한, 설정 정보 등이 저장되어 있다.
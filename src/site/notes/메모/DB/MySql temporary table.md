---
{"dg-publish":true,"permalink":"//db/my-sql-temporary-table/"}
---

#### 특징

- 특정 세션에서만 유효한 테이블 
- **세션 생명 주기와 동일**
- 주로 **복잡한 쿼리를 효율적으로 처리**하거나 **일시적으로 데이터를 저장**할 때 사용
- 임시 테이블 셍성한 세션에서만 사용 가능


#### 명시적 생성/삭제
```sql

CREATE TEMPORARY TABLE temp_table (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);


DROP TEMPORARY TABLE IF EXISTS temp_table;

// 세션의 생명주기와 같으므로 세선 종료시 자동 삭제된다. 
```



#### 묵시적 생성
- MySQL이 내부적으로 복잡한 쿼리를 처리하기 위해 임시 테이블을 생성하는 경우
  **(join, oder by, distinct) 최적화 시에** 
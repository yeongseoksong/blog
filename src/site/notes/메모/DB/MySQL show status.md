---
{"dg-publish":true,"permalink":"//db/my-sql-show-status/"}
---

```sql
flush status

show status lke 'handler_read%'
```

|                            |                                                 |
| -------------------------- | ----------------------------------------------- |
| Variable Name              | Comment                                         |
| Handler_commit             | 커밋 수                                            |
| Handler_delete             | 행을 삭제 하기 위한 요청 횟수                               |
| Handler_discover           | NDBCLUSTER 스토리지 엔진에 요청하여 테이블 이름을 요청한 횟수         |
| Handler_external_lock      | external_lock() 호출한 횟수, 일반적으로 테이블 액세스 시작과 끝에 발생 |
| Handler_mrr_init           | 테이블 액세스에 다중 범위 읽기 구현을 사용한 횟수                    |
| Handler_prepare            | 2단계 커밋 작업을 준비한 횟수                               |
| Handler_read_first         | 인덱스의 첫 번째 키 값을 패치(fetch)한 횟수                    |
| Handler_read_key           | 단일 행의 인덱스 키 값을 읽은 횟수.                           |
| Handler_read_last          | 인덱스의 마지막 키를 읽은 횟수                               |
| Handler_read_next          | 인덱스의 후속 행의 키를 읽은 횟수                             |
| Handler_read_prev          | 인덱스의 이전 행의 키를 읽은 횟수                             |
| Handler_read_rnd           | 고정된 위치의 특정 행을 읽은 횟수                             |
| Handler_read_rnd_next      | 고정된 위치의 특정 행에 대한 후속 행 읽기 횟수                     |
| Handler_rollback           | 스토리지 엔진이 롤백 요청을 받은 횟수                           |
| Handler_savepoint          | Savepoint를 요청한 횟수                               |
| Handler_savepoint_rollback | Savepoit 지점으로 롤백을 요청한 횟수                        |
| Handler_update             | 행을 업데이트 하기 위한 요청 횟수                             |
| Handler_write              | 행을 삽입 하기 위한 요청 횟수<br>                           |
- Handler_read_first의 값은 **index full scan** 발행한 횟수와 동일하다. 
- Handler_read_next 는 인덱스 키 순서에 따라 다음 행을 읽은 횟수로 인덱스를 스캔하거나 제한적으로 범위 검색이 발생할 때 기록된다. Handler_read_first 수치가 예상보다 높게 나타난다면index full scan을 발생시키는 쿼리가 있는지 확인하여 적절하게 인덱스를 사용할 수 있도록 튜닝해야 한다.
- Handler_read_rnd는 테이블 스캔이나 인덱스 범위 스캔이 발생할 때 데이터 파일의 고정된 위치의 행을 읽은 횟수로 대상 테이블에 적절한 인덱스가 없는 경우 발생할 수 있다. 또한 인덱스를 사용하지 않는 조인이 발생하거나 정렬 연상이 발생한 경우도 증가하기 때문에 이 수치가 높다면 요청 쿼리에 대해 적절한 인덱스가 있는지 확인 해야 한다
- select 쿼리에서 handler_write 증가는 임시 테이블 사용을 의미



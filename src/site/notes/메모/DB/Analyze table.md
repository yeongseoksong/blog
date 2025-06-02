---
{"dg-publish":true,"permalink":"//db/analyze-table/"}
---



- 인덱스가 존재하던 테이블에 있는 데이터를 모두 비우고 다른 데이터들을 넣어 주었을 때 `show indexes from table_name` 쿼리시 인덱스의 정보가 (Cardinality) 갱신되지 않았다.
- 이러할 때 `analyze table table_name` 으로 optimize가 사용하는 통계 정보를 갱신 해주어야 한다.
- InnoDB에서 통계정보가 자동으로 갱신되는 경우는 아래와 같다.
   - 전에 인덱스 통계 정보를 갱신한 후 테이블의 전체 행수의1/16이 갱신된 경우  
   - 전에 인덱스 통계 정보를 갱신한 후 20억행 이상이 갱신된 경우
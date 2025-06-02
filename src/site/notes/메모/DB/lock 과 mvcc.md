---
{"dg-publish":true,"permalink":"//db/lock-mvcc/"}
---



**트랜잭션 모델의 한계 second lost update**
- repeatable read  격리 수준은 undo log 를 사용해 동시성을 해결한다.
- 그러나, 이는 **second lost update** 문제를 세밀하게 제어할 수 없다.
	- *첫 번째로 시작한 트랜잭션이 무조건 commit이 인정되게 할 수 없음을 의미.*
- mvcc 특성상 **마지막 커밋만 인정하기** 방식이 적용되게 된다. 즉, 첫번째 커밋된 내역이 분실되게 된다. 
	- *undo log 에서 읽기 경우 tx 번호를 고려해 값을 가져오지만, 테이블의 실제 데이터는 commit 한 시점의 tx 가 결정한다. 그래서 tx 번호가 높더라도 이후에 커밋되면 실제 저장되는 테이블의 값은 tx 번호가 높은 쿼리의 결과 값이다. *
- 격리 수준을 **serializable**로 올리면 **second lost update** 문제가 발생하지 않는다.

**Serializable 은 교착상태를 유발할 수 있음**
- **serializable** 은 읽기를 수행할 때에 s-lock 을 걸고 쓰기 시 당연히 x-lock 을 적용한다.
- **s-lock 과 x-lock 은 동시에 적용될 수 없다는 점을 인지해야 한다.**
- s-lock 걸려있는 레코드에 x-lock 을 걸기 위해서는 기존의 s-lock 을 x-lock으로 변경 해야 한다.
	- tx1 과 tx2  가 레코드 A 에 s-lock 을 걸고 x-lock 으로 변경하는 상황이라 가정하자.
	- 만약 A 에 tx1,tx2 의 s-lock 이 존재할 때 x-lock 을 걸기 위해 다른 s-lock 이 해제됨을 기다려야 한다.
	- 이는 tx1, tx2 의 교착상태를 유발한다.



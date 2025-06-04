---
{"dg-publish":true,"permalink":"//java/connection-pool-jndi/"}
---

#### JNDI
- java naming directory interface 의 약자로 필요한 자원을 key/value 쌍으로 저장해 작업 필요한 자원에 키로 접근한다.
	- 서블릿에서 `getParameter()` 사용 시 
	- HashMap, HashTable
	- DNS 에서 도메인 네임의 IP 주소를 가져올 때
- **톰캣 컨테이너**가 **connection pool 객체**를 생성할 경우에도 JNDI 에 이를 등록한다.
- **WAS** 가 DB 작업 수행시 JNDI 에 KEY 를 사용해 접근 한다. 
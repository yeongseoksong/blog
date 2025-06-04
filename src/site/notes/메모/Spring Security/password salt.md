---
{"dg-publish":true,"permalink":"//spring-security/password-salt/"}
---


#### 개요
---
- salt 를 사용하지 않을 경우 rainbow table 에 의해 DB 가 탈취 당할경우 해커가 비밀번호를 상수 시간에 알아낼 수 있다.
- 그래서 pwd 의 hash 값을 저장할 때 salt 를 추가하여 hash 하고, 테이블에 salt 값 또한 저장한다.

| user  | pwd (plain text + salt) | salt   |
| ----- | ----------------------- | ------ |
| userA | bvcxbcxv                | asdasd |
| userB | xcvbxdsf                | gsdfe  |
| userC | cxvbxcvb                | cxvxcv |
- 무작위 salt 값으로 유저가 같은 pwd 를 사용하더라도 데이터 베이스에서는 다른 pwd 해시 값으로 저장된다.
- salt 값을 고정 값으로 사용할 수도 있으나 그러면 중복되는 비밀번호가 생기고 salt 가 유출 될 경우 해커는 브루트포스 공격으로 여러 유저의 비밀 번호를 한번에 알아낼 수 있다.


#### 공격 유형 1 . 웹에서 브루트 포스
---
- 웹에서 브루트포스 요청하는 경우
- salt 값은 백엔드 내부에서 항상 불러와서 사용하므로 salt값이 의미 없어진다.
- **복잡한 비밀번호를 요구하거나 로그인 횟수를 제한하는 방법으로 방어할 수 있다**

#### 공격 유형 2 . DB 가 유출된 경우
---
- DB 가 유출 됐을 때 해커가 해당 유저의 salt 값을 알기 때문에 원래의 pwd 만 브루트 포트로 알아낼 수 있다.
- pwd = h(plain text + salt), 
  userA 는 bvcxbcxv =h( plain text + asdasd) 
- sha-256 해시는 ms 단위로 값을 얻을 수 있기 때문에 해커가 브루트 포스로 모든 경우의 수를 대입하여 비밀번호를 알아낼 수 있다.
- **이를 해결하기 위해 의도적으로 해시하는데 시간과 컴퓨팅 자원이 많이 드는 해시 알고리즘을 사용한다.** 
  (브루트포스로 모든 값을 넣어보는데 10억년이 걸리게하는 방식)
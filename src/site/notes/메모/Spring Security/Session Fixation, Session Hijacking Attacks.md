---
{"dg-publish":true,"permalink":"//spring-security/session-fixation-session-hijacking-attacks/"}
---

## Session Hijacking Attacks
---

- (공공장소)브라우저의 쿠키에 저장된 세션ID가 탈취 당하는 상황 
- 혹은 네트워크 경로 상에서 세션 ID 를 탈취 당하는 상황
#### 해결 방안
- **Https** 프로토콜로 네트워크 상에서 세션 ID 를 훔칠 수 없게 한다
- 세션 아이디를 짧게 해 public 환경 브라우저에서 세션이 탈취 당했더라도 다른 사용자가 해당 세션을 사용하지 못하게 한다.

## Session Fixation
---
![Session Fixation.png](/img/user/images/Session-Fixation.png)
1. 해커 가 web server 에 로그인해 session 을 획득한다.
2. 피해자에게 1. 에서 획득한 **session id** 를 포함한 url 을 전송해 클릭을 유도한다. (email)
3. 피해자의 브라우저에 session id 인 쿠키가 존재하지 않으므로 로그인을 하도록 요청한다.
4. 피해자의 정보로 로그인을 해 서버 내 **session id 에 피해자 정보가 매핑** 된다.
5. 해커는 **session id 를 통해 피해자의 개인정보 등을 획득**할 수 있게된다.

- web server 의 취약점으로 인해 발생한다.

## Spring Security 에서의 Session Fixation 방어
---
#### spring security 는 기본적으로 3가지 전략을 사용해 이를  방지한다.

- **changeSessionId**
	- 누군가 무작위 세션을 사용해 login 시 최종 사용자가 인증되면 세션 id만 변경하고 매핑된 세션 정보는 그대로 이전한다.
	- 예를들어 해커가 전달한 session id 에 name = "hacker" 로 서버에 매핑되어 있었다면 피해자는 새로운 session id 는 발급 받을 수 있지만, 매핑되어 있는 정보는 name= "hacker" 이다.
	- **즉,changeSessionId는 session id 의 값만 변경한다.**
	- 기본 값이다.
- **newSession**
	- session id 만 변경하는 것이 아닌 session을 새로 만든다.
- **migrateSession**
	- session 을 새로 생성하지만 기존 session id 에 매핑된 값을 복사한다.
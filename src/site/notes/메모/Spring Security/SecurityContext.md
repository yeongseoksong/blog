---
{"dg-publish":true,"permalink":"//spring-security/security-context/"}
---




![seruciryContext.png](/img/user/0.%20%EC%9D%B4%EB%AF%B8%EC%A7%80/seruciryContext.png)

#### SecurityContextHolder
- **SecurityContext** 를 관리하며 crud 기능을 제공
	- `MODE_THREADLOCAL` : 기본 설정으로ThreadLocal 을 사용해 현재 쓰레드에 관련된 보안 컨텍스트를 저장
	- `MODE_INHERITABLETHREADLOCAL` : 위와 유사하나 `@Async` 로 비동기 메서드 호출 시 현재 스레드의 **SecurityContext**를 복사한다.
	- `MODE_GLOBAL` : 시스템 전체에서 하나의 **SecurityContext**에 접근한다. (ex: 동시 사용자가 한명인 데스크톱 어플리케이션)



#### SecurityContext
- 인증 정보 (`Authentication`)를 저장하는 **컨테이너 객체**.

#### Authentication
- 인증된 사용자에 대한 **세부 정보**를 표현하는 인터페이스
	- `principal`: 사용자 정보(일반적으로 `UserDetails` 객체).
	- `credentials`: 비밀번호 또는 인증에 필요한 자격 증명.
	- `authorities`: 권한 리스트.
	- `details`: 추가적인 인증 관련 정보(IP, 세션 정보 등).
	- `isAuthenticated`: 인증 상태 여부.




#### Controller 에서 사용자 인증 정보를 확인하는 방법

**SecurityContextHolder 사용**
전략에 따라 **SecurityContext** 를 불러와 사용
```java
@RequestMapping("/dashboard")  
public String displayDashboard(Model model) {  
    Authentication authentication = SecurityContextHolder.getContext().getAuthentication();  
    if(null != authentication) {  
        model.addAttribute("username", authentication.getName());  
        model.addAttribute("roles", authentication.getAuthorities().toString());  
    }    return "dashboard.html";  
}
```

**Authentication 파라미터 사용**
spring 에서 내부적으로 **SecurityContextHolder**를 사용한다.
```java

@RequestMapping("/dashboard")  
public String displayDashboard(Model model,Authentication authentication) {  
    if(null != authentication) {  
        model.addAttribute("username", authentication.getName());  
        model.addAttribute("roles", authentication.getAuthorities().toString());  
    }    return "dashboard.html";  
}
```
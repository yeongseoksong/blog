---
{"dg-publish":true,"permalink":"//spring-security/csrf/"}
---


#### CSRF 과정
1. 사용자가 naver.com 에서 받은 cookie 를 브라우저에 가지고 있다.
   *(쿠키는 도메인 별로 브라우저에 저장된다.)*
2. 사용자가 새로운 탭을 열어 해커가 관리하는 사이트에 접속
3. 접속시 naver.com 의 특정 경로에 요청을 보내는 script 를 숨겨둔다.
4. 사용자 브라우저는 이를 실행하고 자격증명으로 naver.com 의쿠키를 같이 보내 정상적으로 요청이 네이버 백엔드로 도착하고 처리 될 수 있다.

#### 해결책
- **Referer Check**
- **CSRF Token**
	- 서버에서 요청이 해커의 사이트에서 왔는지 확인할 수 있으면 된다.
	- 세션마다 csrf token (xsrf token) 을 쿠키로 발급한다.
	- 서버에서는 http request 의 헤더 값 **(X-CSRF-Token)** 에 **csrf token** 값을 넣어줄 것을 요구하는데, 쿠키에 java script 로 접근 하기위해서는 쿠키의 도메인이 java script 가 실행되는 웹 페이지의 도메인과 같아야 한다. **(Same-Origin Policy)** 
	- 따라서, 해커의 페이지에서 csrf 쿠키와 인증 쿠키를 요청하는 스크립트를 실행해도 서버에서 csrf 헤더가 존재하지 않으므로 차단된다.


>**쿠키 설정**:
 CSRF 토큰을 저장하는 쿠키는 `HttpOnly`를 사용하지 않아야 클라이언트 측에서 읽을 수 있다.
 반대로 인증 쿠키는 `HttpOnly` 속성을 설정해 JavaScript로 접근하지 못하도록 해야 한다.


#### SPRING SECURITY CSRF

```java
CsrfTokenRequestAttributeHandler csrfTokenRequestAttributeHandler = new CsrfTokenRequestAttributeHandler(); // request 헤더의 csrf 토큰 값을 읽어 attribute 에 저장수행  
http  
        .securityContext(contextConfig -> contextConfig.requireExplicitSave(false)) // form login을 사용하지 않고 HttpBasic 으로 인증을 하는 시나리오이기 때문에  
        // 세션을 생성하고 저장을 명시한다.  
        .sessionManagement(sessionConfig -> sessionConfig.sessionCreationPolicy(SessionCreationPolicy.ALWAYS)) //인증 여부에 상관없이(익명 사용자) 항상 세션 생성  
        
	     .csrf(csrfConfig->csrfConfig  
                .ignoringRequestMatchers( "/contact","/register") // csrf 인증을 수행하지 않을 경로 지정  
                .csrfTokenRequestHandler(csrfTokenRequestAttributeHandler)  // CsrfTokenRequestAttributeHandler 적용  
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())) // cookie 설정을 httpOnly false 로 설정해 브라우저에서 js 로 쿠키 값(csrf token)을 읽을 수 있게한다.

  
        .addFilterAfter(  
                new CsrfCookieFilter(),  
                BasicAuthenticationFilter.class)   //BasicAuthenticationFilter.class 수행 이후 CsrfCookiFilter 커스텀 필터가 수행 된다.
        .requiresChannel(rcc -> rcc.anyRequest().requiresInsecure()) // Only HTTP  
        .authorizeHttpRequests((requests) -> requests  
                .requestMatchers("/myAccount", "/myBalance", "/myLoans", "/myCards", "/user").authenticated()  
                .requestMatchers("/notices", "/contact", "/error", "/register", "/invalidSession").permitAll());  
http.formLogin(withDefaults());  
http.httpBasic(hbc -> hbc.authenticationEntryPoint(new CustomBasicAuthenticationEntryPoint()));  
http.exceptionHandling(ehc -> ehc.accessDeniedHandler(new CustomAccessDeniedHandler()));  
return http.build();
```


- **CookieCsrfTokenRepository** : 사용자의 csrf 쿠키로부터 값을 가져와 DeferredCsrfToken 초기화
  *사용자가 서버에 이상한 쿠키를 넣는 상황은 어떻게 되는지 생각했었는데, csrf 방어는 애초에 사용자가 해커의 사이트에서 원하지 않는 동작을 하게되어 발생하므로 쿠키가 위조될 일이 없다. 해커가 쿠키를 발급한다 하더라도 쿠키 도메인이 다르기도 하다.*


**CsrfFilter.java**
![csrf filter logic.png](/img/user/0.%20%EC%9D%B4%EB%AF%B8%EC%A7%80/csrf%20filter%20logic.png)
1. CSRF 토큰을 헤더에 등록하지 않는"GET", "HEAD", "TRACE", "OPTIONS" 메서드는 `requireCsrfProtectionMatcher`  (false) 에 의해 다음 코드로 넘어간다.
2. CSRF 에 보호 받는 api 에서 수행되며 헤더의 토큰 값을 비교해 일치 하지 않으면 에러를 발생 시킨다.
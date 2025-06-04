---
{"dg-publish":true,"permalink":"//spring-security/cors/"}
---


#### CORS 동작

브라우저는 기본적으로 다른 출처로 api 요청 시 해당 서버에  **preflight** 요청을 보낸다.
**preflight** 요청을 통해 백엔드 서버의 **cors** 옵션을 확인해 **Acess-control-Allow-Origin** 의 허용된 출처를 확인한 후에 응답을 반환한다.
*cors 설정이 안되어 있더라고 백엔드 서버는 요청을 처리한다. 그러나 브라우저에서 응답을 차단한다. *


#### spring security cors
```java
http
.cors(corsConfig -> corsConfig.configurationSource(new CorsConfigurationSource() {  
    @Override  
    public CorsConfiguration getCorsConfiguration(HttpServletRequest request) {  
        CorsConfiguration config = new CorsConfiguration();  
        config.setAllowedOrigins(Collections.singletonList("http://localhost:4200"));  
        config.setAllowedMethods(Collections.singletonList("*"));  
        config.setAllowCredentials(true);  
        config.setAllowedHeaders(Collections.singletonList("*"));  
        config.setMaxAge(3600L);  
        return config;  
    }}))
```






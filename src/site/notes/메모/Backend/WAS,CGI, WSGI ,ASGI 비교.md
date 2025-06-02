---
{"dg-publish":true,"permalink":"//backend/was-cgi-wsgi-asgi/","title":"WAS,CGI, WSGI ,ASGI 비교","tags":["Backend"]}
---


## 0. WAS (web application server)

동적 컨텐츠를 제공하기 위하여 탄생했다. (WAS =Web Sever + Web Container)

>“**웹 컨테이너(Web Container**)” 혹은 “**서블릿 컨테이너(Servlet Container)**”라고도 불린다.
>Container : JSP, Servlet 실행시킬 수 있는 소프트웨어를 말한다. WAS는 JSP, Servlet 구동 환경을 제공한다)

> **Was 는 스레드 풀을 사용한다** 
> [[메모/OS/MultiProcessing Vs MultiThreading\|MultiProcessing Vs MultiThreading]]

---
[참고] 웹 서버 아키텍쳐
https://waspro.tistory.com/331
---

## 1. CGI (common gateway interface ~2003)
---

![Common-Gateway-Interface.webp](/img/user/images/Common-Gateway-Interface.webp)
**공용 게이트웨이 인터페이스**는 서버 프로그램에서 다른 프로그램을 불러내고, 그 처리 결과를 클라이언트에 송신하기 위한 규격을 정의한것을 의미한다. ( 프로토콜 )

CGI 실행 절차는 아래와 같다.

>** 1) Request -> 2) Web Server (apache , nginx  ,ms iis) -> 3) 웹 서버가 프로그램 실행 ( 프로세스 생성)**


> CGI는 Request 수신 시마다 Apllication 프로세스를 다시 실행하여, 메모리 적재 시간 소요 등의 문제 발생한다는 단점이 있다. ( CGI는 요청시마다 프로세스를 생성하여 커널 리소스 사용, 반납 )

 Web Sever + Cgi = WAS 와 같이 동작할 수 있으나 쓰레드를 기반으로 동작하는 WAS 가 더 효과적이라 볼 수 있다. 
## 2. WSGI  (web server gateway interface 2003~)
---
Request 마다 새로운 프로세스를 생성하는 CGI의 단점을 보완하기 위해서 만들어졌다.  ( 파이썬에서 사용되는 개념이다.)

Wsgi 실행 절차는 아래와 같다.

> 1) Request -> 2) Web Sever ->3) WSGI (Gunicorn) middleware -> WSGI Web Framework(Flask,django)

## 3. ASGI (async server gateway interface 2019~)
![1_9YVDOeD0H2nzNHG2BTPAog.webp](/img/user/images/1_9YVDOeD0H2nzNHG2BTPAog.webp)
WSGI에 호환성을 갖으며, 요청을 비동기로 처리할 수 있는 인터페이스 이다. Middleware 로 uvicorn 이 있다.


---
{"dg-publish":true,"permalink":"//backend/uvicorn-gunicorn/","title":"uvicorn , gunicorn 이란","tags":["python","fastapi"]}
---


WSGI ,ASGI 는 Pyhon에서 사용되는 CGI의 일종으로 Web Server 는 여러 언어들의 다양한 요청을 이해할 수 있도록 공통된 규칙으로 변환하기 위한 규약이다.

[참고] [[메모/Backend/WAS,CGI, WSGI ,ASGI 비교\|WAS,CGI, WSGI ,ASGI 비교]]

![Pasted image 20231129135028.png](/images/Pasted%20image%2020231129135028.png)

| 규약 | 구현 미들웨어 | 동작방식 |
| ---- | ------------- | -------- |
| ASGI | uvicorn       | async    |
| WSGI | gunicorn      | sync     |

 ### 0. Gunicorn
---
![Pasted image 20231130130604.png](/images/Pasted%20image%2020231130130604.png)
Gunicorn "Green Unicorn" 으로 Python WSGI(Web Server Gateway Interface, 파이썬의 WAS) HTTP Server로 Ruby의 Unicorn 프로젝트에서 이식된 prefork 방식의 웹 서버이다.


> pre-fork  : 미리 자식 프로세스를 여러개 띄어서 처리한다. 
> [참고] [[메모/기타/prefork 방식이란\|prefork 방식이란]]

*node의 pm2와 같은 프로세스 매니저 역할을 한다.*
### 1. uvicorn
---
Uvicorn은 ASGI 서버이며, 초고속 ASGI 서버에 본질적으로 필요한 것들만을 제공한다. Uvicorn은 uvloop와 httptools를 활용하여 빠른 HTTP 요청 처리를 가능하게 한다. 이는 Python에서 웹 서버를 만드는 과정에서의 주요 병목현상인 네트워크 I/O 처리를 최적화하는 것을 목표로 하고 있다.

> uvicorn 만으로는 웹 애플리케이션을 확장하거나 여러 작업자(worker)를 통해 애플리케이션을 구동하는 것이 어렵다. 또한 uvicorn은 기본적으로 하나의 작업자에서만 실행되므로, 여러 CPU 코어를 활용하는 등의 병렬처리를 위해서는 다른 방법이 필요하다.

	uvicorn main:app --reload

>-- workers 옵션 ??

	uvicorn main:app --workers 4 --reload
위의 명령어로 uvicorn 을 사용하여 프로세스 관리를 해줄 수 있다 그러나 공식 문서에서는 추천하지않는다.

>Nevertheless, as of now, Uvicorn's capabilities for handling worker processes are more limited than Gunicorn's. So, if you want to have a process manager at this level (at the Python level), then it might be better to try with Gunicorn as the process manager.
### 2. gunicorn + uvicorn
---
![Pasted image 20231130141235.jpg](/images/Pasted%20image%2020231130141235.jpg)
Uvicorn은 단일 프로세스로 비동기 처리가 가능하지만, 결국 단일 프로세스라는 한계가 있기 때문에 처리량을 더 늘리기 위해서는 멀티 프로세스를 활용해야 한다.
gunicorn과 uvicorn을 함께 사용하면, ASGI 애플리케이션을 여러 작업자로 분산하여 처리하고, 동시에 다수의 HTTP 요청을 빠르게 처리할 수 있다. 

Gunicorn과 Uvicorn을 함께 사용하는 방법은 다음과 같다.

	gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app


> window에서 gunicorn 사용 불가능하다.


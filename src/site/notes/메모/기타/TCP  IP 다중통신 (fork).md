---
{"dg-publish":true,"permalink":"///tcp-ip-fork/","title":"TCP IP 다중통신 (fork)","tags":["cs","network"]}
---


### 0.  기본적인 서버 통신 방식 (다중통신 x)
---

![Pasted image 20231130105006.png](/img/user/images/Pasted-image-20231130105006.png)

Tcp/ip 통신 에서 accept 시 client 와 sever 사이에 데이터 전송을 위한 소켓이 하나 더 생성된다 .그렇기 때문에 Listening Queue에서 클라이언트 정보가 쌓이게되더라도 연결되어있는 connecting socket이 끊기기 전까지 다른 클라이언트와 연결할 수 없다.

*listening socket은 client들의 정보를  listening queue로 넣어주는 역할만 수행한다 !*

각 요청이 0.1 초 씩 수행된다 하더라도 listening의 100번쨰에 있는 client는 50초나 기다려야한다.

### 1. 어떻게 이를 해결할 것인가?
---
#### 1.a. 클라이언트 접속시 마다 Fork()

![Pasted image 20231130105910.png](/img/user/images/Pasted-image-20231130105910.png)

단순하게 생각하면 각 요청마다 fork 된 서버가 존재해서 client와 connecting socket을 연결하면 된다.

그러나 우리가 웹프레임워크를 사용하여 환경을 구축할때, 자식 프로세스의 포트까지 설정해주지 않더라도 정상적으로 다중통신이 가능하다.
우리는 부모 서버의 포트번호만 설정해준다. 즉 부모서버의 Listening Socket만 이용가능하다는 것이다.

이를 해결하기 위해서는 아래와 그림 같은 구조를 사용해야 한다.

![Pasted image 20231130111047.png](/img/user/images/Pasted-image-20231130111047.png)

Client들의 요청을 받는 Listening 소켓이 부모 프로세스에 존재하고 부모 프로세스는 accept() 시에 fork(), 자식 프로세스의 Connecting Socket과 연결되도록 하면 된다.

![Pasted image 20231130111355.png](/img/user/images/Pasted-image-20231130111355.png)

[출처] https://www.crocus.co.kr/462



---
#### 1.b. prefork() 방식

![Pasted image 20231130112245.png](/img/user/images/Pasted-image-20231130112245.png)

앞서 설명한 방식과 다른점은 요청대기큐에서 client들의 정보를 가져올때 부모 프로세스가 수행 하는것이 아닌 , 자식 프로세스가 부모 프로세스의 binding된 listening socket 을 가지고 있고 자식 프로세스가 요청대기 큐에 접근하는 방식이다.
요청대기큐에 자식프로세스들의  race condition 문제는 os 에서 관리해 처리한다.( mutex , semaphore )  
[출처]https://psyhm.tistory.com/51
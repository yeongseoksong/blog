---
{"dg-publish":true,"permalink":"///prefork/","title":"prefork 방식이란"}
---


![](https://blog.kakaocdn.net/dn/LVJBF/btrH31QvAMH/CsqIQw7dmiqAnUy4WOfCxk/img.png)

prefork 방식은 클라이언트의 요청을 처리하기 전에 미리 **fork() 시스템 콜**을 사용해 자식 프로세스 풀을 만들어. 그 후에 각 자식 프로세스마다 클라이언트의 요청을 배정받아 처리하는 구조다.


#### 참고
---
> Process Pool  vs Thread Pool  ??
> [[메모/OS/MultiProcessing Vs MultiThreading\|MultiProcessing Vs MultiThreading]]


> 서버 프로그램은 어떻게 클라이언트의 요청들을 동시에 처리할까 ?
[[메모/기타/TCP  IP 다중통신 (fork)\|메모/기타/TCP  IP 다중통신 (fork)]]



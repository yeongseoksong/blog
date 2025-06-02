---
{"dg-publish":true,"permalink":"///wsl/"}
---

#### WSL 메모리 제한
---
경로 : C:/Users/사용자명 로 이동한후 .wslconfig 파일을 아래와 같이 생성한다.


```   
memory=4GB
processors=1 
swap=4GB
localhostForwarding=true

```

>localhostForwarding을 해주면 wsl 주소가 아닌 윈도우 주소를 통해서 접속 가능해진다.

![image-20220531152017317.webp](/img/user/images/image-20220531152017317.webp)
스왑메모리 권장 설정양이다.  4gb로 메모리를 설정 할 것이니 swap 도 4gb로 설정 한다.

설정이 완료 됐으면 wsl 을 종료후에 다시 실행한다.

```bash
wsl --shutdown
wsl
```


#### 도커 볼륨 바인딩

---
wsl 쉘에서 윈도우 파일로 경로를 이동한후에 다음 명령어를 입력한다

```bash
docker run -it -d -p 3000:3000 --name test2 -v "$PWD":/home/react node:16
```
컨테이너 접속
```bash
 docker exec -it test /bin/bash
```
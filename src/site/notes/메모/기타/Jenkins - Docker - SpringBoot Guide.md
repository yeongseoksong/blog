---
{"dg-publish":true,"permalink":"///jenkins-docker-spring-boot-guide/"}
---



```table-of-contents

style: nestedList

minLevel: 0

maxLevel: 0

includeLinks: true

debugInConsole: false

```
## 0. Preview
#### 기초 버전

![jenkin pipeline preview.png](/img/user/images/jenkin-pipeline-preview.png)
![젠킨스 spring boot 기본 아키텍처.png](/img/user/images/젠킨스-spring-boot-기본-아키텍처.png)

#### 최종 버전

![젠킨스 최종 파이프라인.png](/img/user/images/젠킨스-최종-파이프라인.png)

![젠킨스 최종 아키텍처.png](/img/user/images/젠킨스-최종-아키텍처.png)


![젠킨스 최종 네이버 클라우드 서버.png](/img/user/images/젠킨스-최종-네이버-클라우드-서버.png)



## 1.  apt install ,  update

```bash
sudo apt update
sudo apt upgrade
```

## 2. Docker install 
```bash
sudo wget -qO- https://get.docker.com/ | sh
```


## 3.  Jenkins install
#### 3.1.  jenkins Image lts version pull

```bash
docker pull jenkins/jenkins:lts
docker images

```
#### 3.2. excute jenkins container
```bash
docker run -d -p 8080:8080 -p 50000:50000 -v /jenkins:/var/jenkins -v /home/ubuntu/.ssh:/root/.ssh -v /var/run/docker.sock:/var/run/docker.sock --name jenkins -u root jenkins/jenkins:lts
```
- -p : 호스트 머신과 port 매핑
- -v : 호스트 머신과 volume 매핑  
- --name : 컨테이너 이름 지정


> **docker.sock** 
> Docker CLI 가 dockerd daemon 에 명령어를 전달하는 통로 역할을 하는 파일이다.
> 여기서는 jenkins 에서 호스트 머신의  docker 데몬을 사용해 docker  이미지를 빌드해야 하기 때문에  볼륨 마운트를 수행, 이를 **Docker in Docker (DID)** 라 한다.



## 4.  무료 도메인 설정
- https://xn--220b31d95hq8o.xn--3e0b707e/  로 접속
- 고급설정 > IP연결(A)  > 공인 ip 입력
- 도메인주소:8080 접속 후 jenkins admin page 확인


## 5. 네이버 클라우드 ACG 포트 허용



## 6. Github,  Jenkins 연동


#### 6.1. ssh 키 생성

Jenkins Container를 생성할 때 "/home/ubuntu/.ssh:/root/.ssh"로 .ssh 디텍도리를 마운트 해놓았기 때문에 Container 밖에서 ssh 키를 생성하면 Jenkins Container와 연결된다.

```bash
$ cd /home/ubuntu/.ssh 

Enter file in which to save the key (/root/.ssh/id_rsa): 
/home/ubuntu/.ssh/id_rsa 입력

$ ssh-keygen
```


#### 6.2. Git hub deploy key 등록

-  Github Repository > Settings > Deploy Keys > Add deploy key로 접속 
- Key 부분에 id_rsa.pub에 들어있는 public key 값을 넣어준다. 

```bash
$ cd /home/ubuntu/.ssh
$ cat id_rsa.pub
 ```

#### 6.3. Git ssh 테스트
- **SSH 에이전트에 개인 키 로드:**
```bash
eval "$(ssh-agent -s)" 
ssh-add ~/home/ubuntu/.ssh/id_rsa`
```
- git clone 이 정상적으로 되는것을 확인 후 디렉터리 삭제
```bash
rm -rf directory
```
#### 6.4. Jenkins Credentials 등록

Jenkins 대시보드 > Jenkins 관리 > Manage Credentials > Credentials에 접속한다.  
Store Jenkins에 Domain이 (global)인 화살표를 눌러 Global credentials (unrestricted)로 이동한다.  
왼쪽 메뉴의 Add credentials를 눌러 credentials를 추가한다.

- Kind  
    SSH Username with private key
- ID  
    github -> 마음대로 지어도 된다. 다만 Pipeline Script 작성 시 credentialsId로 사용되니 식별할 수 있도록 하자.
- Username  
    root (default)
- Private Key  
    Enter directly 체크 -> private key 입력  
    여기서 private key는 Jenkins Server에서 생성한 id_rsa이다. 아래 명령어로 확인 가능하다.
    
    ```bash
    $ cd /home/ubuntu/.ssh
    $ cat id_rsa
    ```
    
    ```
    -----BEGIN OPENSSH PRIVATE KEY-----
    ...
    ... 이와 같은 형태의 key가 private key 입니다 ...
    ...
    -----END OPENSSH PRIVATE KEY-----
    ```
    
- OK를 눌러 키를 생성

>Jenkins가 Docker 컨테이너 내부에서 실행 중일 경우, 외부 리소스(예: GitHub)와의 연결 설정에 추가적인 고려 사항이 필요하다.
Docker 컨테이너 내부에서 GitHub에 SSH를 통해 접근하려면, 컨테이너 내부에 SSH 키와 관련 설정을 올바르 게 구성해야 한다.

```bash
$ docker exec -it  jenkins bash 
$ ssh-keyscan -t rsa github.com >> /root/.ssh/known_hosts
```

#### 6.5. DooD 설정
- jenkins container 에서 호스트 머신의 docker cli 를 사용하기 위해서는 docker.sock 에 대한 권한 수정이 필요하다.
- jenkins 는 root 권한이 없는 사용자로 실행되기 때문에 docker.sock 에 대한 권한을 모두에게 허용하는 작업
```bash
$ cd /var/run
$ ls -l | grep docker
$ chmod 666 docker.sock
$ ls -l |grep docker
```
- 컨테이너 내부에 docker cli 설치
```bash
$ apt-get update
$ apt-get install -y docker.io
```

>**DinD ( docker in docker)** : container 내부에 docker 를 설치한다. docker 제작사에서 권장하지 않는 방법이다.
**Dood ( docker out of docker )** :호스트머신의 docker.socekt 를 mnt 한다. 이때, docker.socket 의 권한을 777로 바꾸어 주거나 docker.so의 Gid를 container user 에 적용해 주어야 한다



## 7. Jenkins 플러그인 설치
- Docker, Docker Pipeline
- Github Integration 

## 8. github webhook 설정
- setting > webhooks > add webhook 
- **payload url** : http://도메인:8080/github-webhook/
- **content type** : application/x-www-form-urlencoded
- 나머지 설정은 기본 값으로 설정한다.

## 9. 프로젝트 설정

####  9.1.  jar 파일 이름 지정
-build.gradle 에 추가한다.
```
tasks.bootJar {  
    archiveBaseName.set('server')   // 파일 이름  
    archiveVersion.set('')     // 버전  
    archiveClassifier.set('')       // 분류자 (필요 없으면 빈 문자열)  
}
```

#### 9.2. Dockerfile 생성
- docker image 를 만들기 위해서는 아래의 dockerfile 이 프로젝트 root 에 존재해야한다.
```dockerfile
# Use a lightweight Java runtime image  
FROM amazoncorretto:17-alpine  
  
# Set the working directory inside the container  
WORKDIR /app  
  
# Copy the built Spring Boot JAR file into the container  
COPY build/libs/server.jar app.jar  
  
# Expose the port your app listens on  
EXPOSE 8080  
  
# Command to run the Spring Boot app  
ENTRYPOINT ["java", "-jar", "app.jar"]
```


#### 9.3. Jenkinsfile 생성
- Jenkins 에서 repository 를 clone 한 이후에 Jenkinsfile 에 작성되어 있는 pipeline 스크립트를 기반으로 작업을 수행한다. 다음과 같다.
```
pipeline {  
    agent any  
  
    environment {  
        imagename = "spring"  
        containerName = 'spring'  
    }  
  
    stages {  
        stage('Prepare') {  
            steps {  
                echo 'Cloning Repository'  
                git url: 'git url',  
                    branch: 'main'  
	      }  
        }  
  
        stage('Build Gradle') {  
            steps {  
                echo 'Building Gradle'  
                dir('.') {  
                    sh './gradlew clean bootjar'  
                }  
            }  
            post {  
                failure {  
                    error 'This pipeline stops here...'  
                }  
            }  
        }  
  
        stage('Build Docker') {  
            steps {  
                echo 'Building Docker Image'  
                script {  
                    // Docker 이미지 빌드  
                    dockerImage = docker.build("${imagename}")  
                }  
            }  
            post {  
                failure {  
                    error 'This pipeline stops here...'  
                }  
            }  
        }  
         stage('Stop Docker Container') {  
                    steps {  
                        echo 'Stopping Docker Container'  
                        script {  
							 def runningContainers = sh(script: "docker ps -a -q --filter name=${containerName}", returnStdout: true).trim()  
                              if (runningContainers) {  
                                  sh "docker stop ${runningContainers}"  
                                  sh "docker rm ${runningContainers}"                         }  
                        }  
                    }  
                    post {  
                        failure {  
                            error 'This pipeline stops here...'  
                        }  
                    }  
                }  
  
        stage('Run Docker Container') {  
            steps {  
                echo 'Running Docker Container'  
                script {  
                    // Docker 컨테이너 실행  
                    dockerImage.run("-d --name ${containerName} -p 80:8080")  
                }  
            }  
            post {  
                failure {  
                    error 'This pipeline stops here...'  
                }  
            }  
        }  
    }  
  
    post {  
        always {  
            echo 'Pipeline completed'  
        }  
    }  
}
```



## 10 . Jenkins Machine, Spring Machine 분리
![젠킨스 최종 아키텍처.png](/img/user/images/젠킨스-최종-아키텍처.png)

- 클라우드 콘솔에서 spring 용 machine 을 생성 한다. 이때 서브넷은 jenkins 와 같게 설정한다.
- ACG 설정에 들어가 outbound 를 모두 열어준다.
- inbound 의 경우 spring 으로 사용하게 될 80 포트를 0.0.0.0/0 으로 열어준다.
- 또, jenkins 에서 해당 서버로 ssh 연결을 해야하므로 22 포트는 jenkins 서버의 내부 아이피 주소 로 허용해준다.

```bash
ssh root@172.16.0.7 
```
- docker 설치  이후 docker 가 잘 동작하는지 확인한다.

#### 10.1. Jenkins 컨테이너에 sshpass 설치

```bash
$ sudo apt-get update 
$ sudo apt-get install sshpass
```


#### 10.2. Docker Plugin 설치

Jenkins 대시보드 > Jenkins 관리 > 플러그인 관리 > 설치 가능 > Docker 검색 > Docker, Docker Pipeline 플러그인 설치 및 재실행

> 앞서 이미 설치 한 플러그인 들이므로 이므로 생략 가능하다.

#### 10.3. Docker Hub Credentials 등록

Jenkins 대시보드 > Jenkins 관리 > Manage Credentials > Credentials에 접속한다.  
Store Jenkins에 Domain이 (global)인 화살표를 눌러 Global credentials (unrestricted)로 이동한다.  
왼쪽 메뉴의 Add credentials를 눌러 credentials를 추가한다.

- Kind  
    Username with password
- Username  
    본인의 Docker Hub ID
- Password  
    본인의 Docker Hub Password
- ID  
    docker-hub -> 마음대로 지어도 된다. 다만 Pipeline Script 작성 시 credentialsId로 사용되니 식별할 수 있도록 하자.


## 10.4 Jenkinsfile 수정


```
pipeline {  
    agent any  
  
    environment {  
        imagename = "spring"                       // Docker 이미지 이름  
        containerName = 'spring'                  // Docker 컨테이너 이름  
        remoteServer = "remote server"               // remote server 내부주소 
        remoteServerUserName = "root"             // 원격 서버 사용자 이름  
        remoteServerPassword = "pwd"  
		dockerHubCred = 'docker'    // Jenkins에 저장된 Docker Hub 자격 증명 ID              dockerHubUserName="dockerhub usename"  
    }  
  
    stages {  
        stage('Prepare') {  
            steps {  
                echo 'Cloning Repository'  
                git url: git url',  
                    branch: 'main'  
            }  
        }  
  
        stage('Build Gradle') {  
            steps {  
                echo 'Building Gradle'  
                dir('.') {  
                    sh './gradlew clean bootjar'  
                }  
            }  
            post {  
                failure {  
                    error 'Gradle build failed. Stopping the pipeline.'  
                }  
            }  
        }  
  
        stage('Build Docker') {  
            steps {  
                echo 'Building Docker Image'  
                script {  
                    // Docker 이미지 빌드  
                    dockerImage = docker.build("${dockerHubUserName}/${imagename}")  
                }  
            }  
            post {  
                failure {  
                    error 'Docker build failed. Stopping the pipeline.'  
                }  
            }  
        }  
  
        stage('Push Docker') {  
            steps {  
                echo 'Pushing Docker Image to Docker Hub'  
                script {  
                    docker.withRegistry('', dockerHubCred) {  
                        dockerImage.push()  // Docker 이미지 푸시  
                    }  
                }  
            }  
            post {  
                failure {  
                    error 'Failed to push Docker image. Stopping the pipeline.'  
                }  
            }  
        }  
  
  
        stage('Deploy to Remote Server') {  
            steps {  
                echo 'Connecting to Remote Server and Deploying Docker Container'  
                script {  
                    // `sshpass`를 사용해 암호 기반으로 SSH 접속  
                    sh """  
                        sshpass -p '${remoteServerPassword}' ssh -o StrictHostKeyChecking=no ${remoteServerUserName}@${remoteServer} "                            docker pull ${dockerHubUserName}/${imagename} &&                            docker ps -q --filter name=${containerName} | grep -q . && docker rm -f ${containerName} || true &&                            docker run -d --name ${containerName} -p 80:8080 ${dockerHubUserName}/${imagename}  
                        "                    """  
                }  
            }  
            post {  
                failure {  
                    error 'Failed to deploy Docker container on remote server. Stopping the pipeline.'  
                }  
            }  
        }  
    }  
  
    post {  
        always {  
            echo 'Pipeline execution completed.'  
        }  
    }  
}
```







# 도커 설치부터 IBM MQ 설치까지 

## openJdk1.8 다운

##### 1. openJdk1.8 다운

```
yum install java-1.8.0-openjdk
```



2.환경변수 등록

> readlink -f /usr/bin/java/usr/lib/jvm/JDK이름/jre/bin/java

```
readlink -f /usr/bin/java/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.242.b08-0.el7_7.ex86_64/jre/bin/java
```



3.환경변수 설정

```
# 3. 환경변수 잡아주기
vim /etc/profile

# 파일 맨 아래에 아래 내용 추가
export JAVA_HOME=/usr/lib/java/openjdk-8u292-b10
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=$CLASSPATH:$JAVA_HOME/jre/lib/ext:$JAVA_HOME/lib/tools.jar

# 적용
source /etc/profile
```



## sts 다운

```
wget https://download.springsource.com/release/STS4/4.13.0.RELEASE/dist/e4.22/spring-tool-suite-4-4.13.0.RELEASE-e4.22.0-linux.gtk.x86_64.tar.gz
```

sts 압축해제

```
tar spring-tool-suite-4-4.13.0.RELEASE-e4.22.0-linux.gtk.x86_64.tar.gz
```







## 도커 설치 과정(CentOS)

- yum-utils 패키지 설치 (yum 명령어 사용하기 위해서)

```
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y yum-utils
```

- 도커 엔진 설치

```
 sudo yum install docker-ce docker-ce-cli containerd.io
```

- 도커 등록

```
 sudo systemctl enable docker
```

- 도커 시작

```
sudo systemctl start docker
```

- 도커 엔진 테스트

```
sudo docker run hello-world
```

출처:https://docs.docker.com/engine/install/centos/



## 도커에 IBMMQ 설치

```
docker pull ibmcom/mq:latest
```

```
docker images
```

```
docker volume create qm1data
```

```
docker run --env LICENSE=accept --env MQ_QMGR_NAME=QM1 --volume qm1data:/mnt/mqm --publish 1414:1414 --publish 9443:9443 --detach --env MQ_APP_PASSWORD=passw0rd ibmcom/mq:latest
```

- 동작중인 도커 확인

```
docker ps
```

- 큐 매니저에 접근하기 위해 해당 명렁어를 사용하여 해당 컨테이너의 설정으로 들어감

```
docker exec -ti <my container id> /bin/bash
```

- 큐 매니저 실행

```
runmqsc <my container id>
```

- 현재 작동중인 MQ확인

```
dspmqver
```

- 작동중인 큐 매니저 확인

```
dspmq
```

- 해당 명령어를 통해서 web 콘솔 주소를 확인하고 해당 주소로 접속하자

```
dspmqweb status
```

- 나가기

```
exit
```



이로서 컨테이너 내부에는 해당 정보의 MQ가 동작하고 있을것이다.

큐 매니저  QM1

큐 DEV.QUEUE.1

채널: DEV.APP.SVRCONN

리스너: SYSTEM.LISTENER.TCP.1 on port 1414

http://www.freekb.net/Article?id=2214




## 도커 설치 과정

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


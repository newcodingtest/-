### 리눅스에서 특정 jdk 버전 path로 설정하기

#### 1.yum으로 jdk설치

버전확인

> java -version

설치 가능한 jdk 리스트 확인

> yum list java*jdk-devel 

원하는 버전 다운

> yum install java-11.0.0-openjdk-devel.x86_64

특정 버전을 path에 잡아주기

> alternatives --config java

![image](https://user-images.githubusercontent.com/57785267/173313815-775c12be-10bb-4a00-81dc-0edc3a8d767e.png)



### 2.tar.gz으로 jdk설치

#### jdk 설치공간 생성하기

> cd /user/local
>
> mkdir workspace
>
> tar -xvf jdk11

#### jdk 설치확인

> cd /user/local/workspace/jdk11/
>
> ./javac -version

#### jdk 설정

##### 1. /etc/profile 백업 , profile.org에 원본 설정을 백업해둔다.

![img](https://blog.kakaocdn.net/dn/k632W/btqD2c3sr8H/Mw2jnNqiDXMU69OpW8RhNk/img.png)



##### 2. vim /etc/profile 을 수정하여 원하는 jdk 경로를 집어넣는다.

![img](https://blog.kakaocdn.net/dn/b05SaX/btqD1NpmOEg/FzTTLirPnh0bqu5Nb3z2mk/img.png)

```
export JAVA_HOME=/usr/local/workspace/jdk1.8.0_251
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/tools.jar
```

![img](https://blog.kakaocdn.net/dn/CHyAY/btqD3HIgDjb/cK2Rksk4IX6tY8k9WZksPk/img.png)



##### 3.source /etc/profile 적용

![img](https://blog.kakaocdn.net/dn/bt4OVt/btqD2bXOSXX/eEKyoMp1RysknZ7JzHtW81/img.png)

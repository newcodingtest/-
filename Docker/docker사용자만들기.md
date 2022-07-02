#### docker 사용자 만들기

> mkdir /app

> groupadd app

> useradd -g app -d /app/docker docker

> passwd docker

> su - docker

> id

#### docker 유저에게 sudo 권한 주기

> chmod u+w /etc/sudoers

> vi /etc/sudoers

> docker ALL=(ALL)    NOPASSWD: ALL
>
![image](https://user-images.githubusercontent.com/57785267/177015209-ccbaf08f-3886-4023-a2aa-550395b1db22.png)

> su - docker

#### docker 계정에서 sudo 계정으로 접근하기 test

> sudo vi /etc/hosts

Docker 에 mariaDB 설치

```
docker pull mariadb
```

컨테이너 실행

```
docker run -d --name mariadb \
-e MYSQL_DATABASE=prod -e MYSQL_USER=yjy \
-e MYSQL_PASSWORD=yjy \
-e MYSQL_ROOT_PASSWORD=1234 \
-P 3306:3306 \
-v /app/maria:/var/lib/mysql \
mariadb
```

/app/maria : 로컬 마리아db 볼륨

/var/lib/mysql: 도커 마리아db 볼륨

도커 재기동시에도 db데이터가 유지되기 위해서

```
-v /app/maria:/var/lib/mysql \
```



```
docker ps
```






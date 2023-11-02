Rabbitmq Docker 실행 명령어**

```
docker run -d --hostname my-rabbit --name some-rabbit -p 8081:15672 -p 5672:5672 -v /home/rabbitmq/data:/var/lib/rabbitmq/ -v /home/rabbitmq/logs:/var/log/rabbitmq/ -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin rabbitmq:3-management
```



**유저 추가**

```
rabbitmqctl add_user guest guest
```



**권한 추가**

```
rabbitmqctl set_user_tags guest administrator
```



**접근 허용범위 추가**

```
abbitmqctl set_permissions -p atm guest "." "." ".*"
```



**큐 생성하기**

```
-rabbitmqadmin -u guest -p guest -V atm declare queue name=q.atm
```



**rabbitmq config 파일 import 하기**

```
rabbitmqadmin import rabbit_a63e5330d5cc_2023-5-16.json
```



 **rabbitmq 초기화**

```
1.rabbitmqctl stop_app
2.rabbitmqctl reset
3.rabbitmqctl start_app
```



**queue 데이터 초기화**

```
rabbitmqadmin --port=15672 --username=guest --password=guest purge -V atm queue name=q.atm
```













**show queue list**

```
rabbitmqctl list_queues
```



**show exchange list**

```
rabbitmqctl list_exchanges
```



**add vrhost**

```
rabbitmqctl add_vhost atm
```



**set  permissions to id about vrhost**

```
rabbitmqctl set_permissions -p atm guest ".*" ".*" ".*"
```



**make a exchange**

```
rabbitmqadmin declare exchange --vrhost=atm name=x.atm type=topic
```



**make a queue**

```
rabbitmqadmin declare queue --vrhost=atm name=q.atm durable=true
```



**make a bining**

```
rabbitmqadmin --vrhost=atm declare binding source=x.atm destination=q.atm routing_key=x.key
```


#### **1.유저 추가**

> rabbitmqctl add_user test test

#### 2.권한 추가

> rabbitmqctl set_user_tags test administrator

#### 3.접근허용 범위 추가

> rabbitmqctl set_permissions -p / test ".*" ".*" ".*"




docker run -d --hostname my-rabbit --name some-rabbit -p 8081:15672 -p 5672:5672 -v /home/rabbitmq/data:/var/lib/rabbitmq/ -v /home/rabbitmq/logs:/var/log/rabbitmq/ -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin rabbitmq:3-management




### 4.rabbitmqctl 로 큐 생성하기

> rabbitmqadmin -u root -p root -V / declare queue name=010


### 5.rabbitmq config 파일 import 하기

> rabbitmqadmin import rabbit_a63e5330d5cc_2023-5-16.json



### 6.rabbitmq 초기화

> 1.rabbitmqctl stop_app <br>
> 2.rabbitmqctl reset <br>
> 3.rabbitmqctl start_app


<br>

### 7.rabbitmq queue 안에 들어있는 데이터 clean 하기

>  rabbitmqadmin --port=15672 --username=root --password=root purge -V/ queue name=002r

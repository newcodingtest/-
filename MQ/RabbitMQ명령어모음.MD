#### **1.유저 추가**

> rabbitmqctl add_user test test

#### 2.권한 추가

> rabbitmqctl set_user_tags test administrator

#### 3.접근허용 범위 추가

> rabbitmqctl set_permissions -p / test ".*" ".*" ".*"




docker run -d --hostname my-rabbit --name some-rabbit -p 8081:15672 -p 5672:5672 -v /home/rabbitmq/data:/var/lib/rabbitmq/ -v /home/rabbitmq/logs:/var/log/rabbitmq/ -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin rabbitmq:3-management

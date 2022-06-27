#### **1.유저 추가**

> rabbitmqctl add_user test test

#### 2.권한 추가

> rabbitmqctl set_user_tags test administrator

#### 3.접근허용 범위 추가

> rabbitmqctl set_permissions -p / test ".*" ".*" ".*"



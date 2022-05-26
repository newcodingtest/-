
|   작업   |                        우분투 명령어                         |                        CentOS6 명령어                        |                        CentOS7 명령어                        |
| :------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|   시작   | [service mysql start](https://zetawiki.com/w/index.php?title=Service_mysql_start&action=edit&redlink=1) | [service mysqld start](https://zetawiki.com/w/index.php?title=Service_mysqld_start&action=edit&redlink=1) | [systemctl start mysqld](https://zetawiki.com/w/index.php?title=Systemctl_start_mysqld&action=edit&redlink=1) |
|   정지   | [service mysql stop](https://zetawiki.com/w/index.php?title=Service_mysql_stop&action=edit&redlink=1) | [service mysqld stop](https://zetawiki.com/w/index.php?title=Service_mysqld_stop&action=edit&redlink=1) | [systemctl stop mysqld](https://zetawiki.com/w/index.php?title=Systemctl_stop_mysqld&action=edit&redlink=1) |
|  재시작  | [service mysql restart](https://zetawiki.com/w/index.php?title=Service_mysql_restart&action=edit&redlink=1) | [service mysqld restart](https://zetawiki.com/w/index.php?title=Service_mysqld_restart&action=edit&redlink=1) | [systemctl restart mysqld](https://zetawiki.com/w/index.php?title=Systemctl_restart_mysqld&action=edit&redlink=1) |
| 상태확인 | [service mysql status](https://zetawiki.com/w/index.php?title=Service_mysql_status&action=edit&redlink=1) | [service mysqld status](https://zetawiki.com/w/index.php?title=Service_mysqld_status&action=edit&redlink=1) | [systemctl status mysqld](https://zetawiki.com/w/index.php?title=Systemctl_status_mysqld&action=edit&redlink=1) |

**mysql**

**1) Global 설정 확인**

```
mysql> show global variables like '%timeout';
```

**2) Session 설정 확인**

```
mysql> show variables like '%timeout';
```

\3. Timeout 시간을 조절하려면 아래와 같이 설정합니다.

**1) Global 설정**

```
mysql> set global interactive_timeout=10;

mysql> set global wait_timeout=10;
```

**2) Session 설정**

```
mysql> set interactive_timeout=10;

mysql> set wait_timeout=10;
```

단, 위와 같은 방법은 mysql 데몬 재시작 시 초기 값으로 돌아갑니다. 

**mysql 시작 시 고정된 값으로 설정하기 위해서는 아래와 같이 my.cnf 파일을 통해 설정합니다.**

```
# vi /etc/my.cnf

[mysqld]

interactive_timeout=10

wait_timeout=10
```

 

#### MySQL 서버의 time out 설정은 크게 3가지가 있습니다. 

#####  mysqld time out 

  \- connect_timeout (bad handshake timeout)   
  \- interactive_timeout (interactive 모드에서 connection time out)   
  \- wait_timeout (none interactive 모드에서 connection time out) 

connect_timeout, interactive_timeout 은 튜닝과 좀 거리가 멀고,실제로 바쁜 서버라면, 
반드시 wait_timeout 을 따로 튜닝하여 설정해줘야 합니다. 

##### [connect_timeout] 

이 설정은 mysqld 와 mysql client 가 연결(connection)을 맺기 위해서mysqld 가 연결 패킷을 기다리는 최대 시간입니다. 
즉 TCP 연결을 맺는 과정(3-way handshake)에서, connect_timeout 동안에도연결 패킷이 들어오지 않으면 연결이 실패(취소가 아님)되고,bad handshake 로 응답합니다. 

 *참고) 
    \- 연결 실패 : 연결 과정중에서 fail 되는 경우 (Aborted_connects)   
    \- 연결 취소 : 연결이 된 상태에서 강제로 close 된 경우 (Aborted_clients) 

다시 말하면 mysqld 와 mysql client 가 TCP 연결을 맺는 최대 시간으로이 시간보다 큰 경우는 모두 Aborted_connects 에 해당됩니다.(단위는 초) 
 연결 실패율(POF) = (   Aborted_connects * 100 / Connections ) 
 연결이 실패되는 경우    
    \- 연결 패킷에 올바른 연결 정보가 없는 경우   
    \- 특정 user 가 권한이 없는 데이터베이스에 접근할 경우   
    \- mysqld 접근 password 가 틀린 경우   
    \- connect_timeout 보다 긴 연결 과정 

연결 실패율(POF)이 높은 경우는, 대부분 권한이 없는 데이터베이스 연결이나, 틀린 password 를 사용할 경우가 많습니다. 
기본값은 대부분 5(초)로 설정되어 있으며, 따로 튜닝할 필요는 없습니다. 
mysqld 의 --warnings 옵션 사용과 xxx.err 파일에 기록됩니다. 



##### [interactive_timeout] 

interactive 모드에서 time out 을 말합니다. 
interactive 모드는 'mysql>' 과 같은 프롬프트 있는 콘솔이나 터미널 모드를말합니다. 
mysqld 와 mysql client 가 연결을 맺은 다음, 다음 쿼리까지 기다리는최대 시간을 의미합니다. 
설정된 interactive_timeout 까지도 아무런 요청(쿼리)이 없으면 연결은취소되고, 
그 이후에 다시 요청이 들어오면 연결은 자동으로 맺어집니다. 

interactive_timeout 안에 다시 요청이 들어오면 wait time은 0으로 초기화됩니다(CLIENT_INTERACTIVE). 

 ERROR 2006: MySQL server has gone away No connection. Trying to reconnect... Connection id:  12002 Current database: xxx 

이와 같은 연결 취소는 Aborted_clients 에 누계되고, wait_timeout 의결과도 함께 포함됩니다 
기본 값은 28800(8시간) 초로 설정되어 있는데 상당히 관대한 설정입니다. 
약 1시간(3600) 정도로 설정하는 것을 권장합니다. 



##### [wait_timeout] 

이 설정은 제일 중요한 파라메터 항목입니다. 
interactive 모드가 아닌 경우에 해당되며,mysqld 와 mysql client 가 연결을 맺은 후, 
다음 쿼리까지 기다리는최대 시간을 의미합니다. 
즉 대부분 PHP 나 C, PERL, python 등등의 API 를 이용한 client 프로그램모드를 말합니다. 
interactive_timeout 과 마찬가지로 wait_timeout 까지 아무런 요청(쿼리)이없으면 연결은 취소되고 
그 결과는 Aborted_clients 에 누계됩니다. 
wait_timeout 안에 다시 요청이 들어오면 wait time 은 0 으로 초기화 됩니다.(SESSION.WAIT_TIMEOUT) 
 연결 취소율(POC) = (   Aborted_clients * 100 / Connections ) 
 연결이 취소되는 경우(강제 종료됨) 
   \- 종료(exit) 되기전 mysql_close() 가 없는 경우   
   \- wait_timeout 이나 interactive_timeout 시간까지 아무런 요청(쿼리)이 없는 경우 

기본 값은 interactive_timeout 과 마찬가지로 28800(8시간) 초로 설정되어있는데, 역시 너무 관대한 설정이라고 할 수 있습니다. 
앞에서 연결 취소율(POC)을 계산해 보면, MySQL 서버가 어느 정도 비율로 강제종료하는지 알 수 있습니다. 
예를 들어 POC 가 1 % 이라면, 100 개의 커넥션당 하나 정도는 mysql_close()없이 강제 종료(exit)되고 있다는 의미입니다. 
이 값이 0 %에 가까울수록 좋습니다. 이 의미는 클라이언트 프로그램에서모두 정상적으로 종료했다는 의미입니다.





##### CentOs 서버 에서 외부아이피로 포트를 열어줘서

##### 등록된 외부아이피 에서 db서버에 접속할수 있게 하기

> mysql -h 192.168.0.241 -u root -p

>  iptables -I INPUT 1 -p tcp --dport 3306 -j ACCEPT

> iptables -I OUTPUT 1 -p tcp --dport 3306 -j ACCEPT


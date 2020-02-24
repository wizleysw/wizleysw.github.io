---
published: true
layout: single
title : "PHP + nginx + mysql @ docker로 게시판 구현하기"
category : dev
comments: false
author_profile : true
tag : PHP
toc : true
---

## Overview

이번 프로젝트의 목표는 크게 3가지로 나눌 수 있을 것 같다.

첫 번째 목표는 웹개발을 해본다는 것이다. PHP는 대학교를 첫 입학했던 시기에는 가장 많이 사용하는 웹 개발 언어였다. 하지만 지금은 React, Django 등의 더 사용하기 쉽고 강력한 프레임웍들이 나왔고 PHP의 개발자들이 단점 보완에 적극적이지 않다는 이유등으로 점차 사용되지 않는 언어가 되어가고 있다. 하지만 기존에 구현된 수많은 사이트는 php를 사용하기 때문에 한 번쯤은 사용을 해보는 것이 좋다고 생각하였다. 

두 번째 목표는 도커를 사용해보는 것이다. 현재 내 머리속의 스케치를 보면 php, nginx, mysql의 조합으로 이번 프로젝트를 진행하게 될 터인데 Docker의 장점을 활용하여 배포시에 환경 세팅에 대한 부분을 덜 수 있으며 가능하면 Docker를 통해 망분리를 진행해보고자 한다. 이 부분은 Docker를 아직 제대로 사용해본 적이 없기 때문에 진행되면서 보완이 될 것이라는 생각이 된다. 

마지막으로 세 번째 목표는 보안의 관점에서 개발을 진행해보는 것이다. 고질적으로 php는 예전부터 보안적인 측면이 약했다고 생각한다. 물론 지금은 많이 발전하여 xss등에 대한 대비도 가능하겠지만 개발을 진행하면서 이런 부분들에 대해서 고민해보는 시간을 가지면 좋을 것 같았다. 

만들고자 하는 게시판의 기본기능은 다음과 같이 요약이 가능할 것 같다.

1. 회원가입 
2. 로그인
3. 글 작성
4. 글 수정
5. 글 삭제 
6. 글 조회
7. 회원탈퇴

위의 모든 작업들은 DB처리가 필요하며 로그인을 위해서는 세션에 대한 구현도 필요하다. 또한 글에 대한 생성/조회/삭제의 과정은 쿼리에 대한 처리의 과정이 될 것이며 그 과정에서 xss, injection, csrf 등의 악의적인 공격이 가능한지를 검증하는 과정이 필요할 것이다. 그저 개발을 진행하는 것보다는 상용된 사이트의 모습을 모방하여 진행을 할 것인데 그 예제로는 PPOMPPU라는 사이트를 사용하게 될 것 같다. 해당 사이트의 자유게시판을 모습을 엇비슷하게 만들면 디자인에 대한 고민의 시간을 줄일 수 있다는 판단때문이다.

## Settings
위에서 언급했듯이 올해의 첫 프로젝트부터 Docker를 활용하기로 맘을 먹었다. 그리고 나는 Mac을 사용하기 때문에 Mac에서 도커를 생성하여 개발을 진행할 것이다. 게시판은 php, nginx, mysql의 조합으로 개발을 진행할 것인데 mysql은 DB로 사용되는 것을 알지만 nginx가 왜 필요한지에 대한 의문이 들었다. 

### nginx

nginx는 동시접속을 효율적으로 처리하기 위해 개발이 된 웹 서버 프로그램이며 동시접속자의 수가 700명을 초과할 경우 서버를 증설하거나 nginx를 도입한다고 한다. nginx는 크게 두 가지 역할을 가지고 있는데 첫 번째 역할이 HTTP 서버이다. Html, CSS, Javascript등의 정적인 정보들을 앞단에서 처리해줌으로써 서버의 과부하를 줄이는데 사용됨을 의미한다. 

두 번째로는 리버스 프록시 서버로서의 역할인데 클라이언트가 웹 서버로 바로 요청을 하는 것이 아닌 nginx서버를 요청을 하고 nginx 서버에서 요청을 전달하도록 하는데, 이 상황에서 nginx가 여러 서버로 분산해서 요청을 전달하는 로드 밸런싱을 위해 사용되기도 한다는 것이다. 이를 통해 서버에 대한 노출이 최소화되며 layer가 한단계 더 추가된다는 장점이 있다. 

또 하나의 특징으로는 비동기 처리 방식을 채택한다는 점을 들 수 있는데, 이로 인하여 1개의 연결당 하나의 쓰레드를 할당하는 방식이 아닌 Event handler가 다수의 연결을 비동기로 처리해 한정된 자원으로 처리가 가능하다. 해당 부분에 대한 설명은 아래의 링크를 참조하면 좋을 것 같다.

[readdir과 readdirSync 성능비교와 비동기 그리고 동기](https://blog.naver.com/jhc9639/221108496101)

그러면 이제 Docker를 설치해야 되는데 mysql, php, nginx를 각각 다른 Container로 받아오는 작업을 할 차례이다. latest 태그를 통해 최신 버전을 받을 수 있지만 개발 환경을 통일하기 위해 모든 컨테이너를 특정 버전을 명시하여 받을 것이다.

### docker container

도커에 대한 간단한 사용법은 아래의 링크를 통해 배우면 좋다.

[초보를 위한 도커 안내서](https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html)
[컴포즈 사용법](https://www.44bits.io/ko/post/almost-perfect-development-environment-with-docker-and-docker-compose)

dockerHub 공식 사이트를 확인해보면 현재 latest의 tag를 확인할 수 있는데 글을 쓰는 현재의 nginx의 latest는 1.17버전인 것을 확인이 가능하다. 

```console
Wizley:~/Project/PHP/Board # docker pull nginx:1.17
1.17: Pulling from library/nginx
bc51dd8edc1b: Pull complete
66ba67045f57: Pull complete
bf317aa10aa5: Pull complete
Digest: sha256:ad5552c786f128e389a0263104ae39f3d3c7895579d45ae716f528185b36bc6f
Status: Downloaded newer image for nginx:1.17
docker.io/library/nginx:1.17
```

마찬가지로 같은 방식으로 php와 mysql등의 필요한 Container를 다운받는다.

```console
Wizley:~/Project/PHP/Board # docker pull php:7.4
7.4: Pulling from library/php
bc51dd8edc1b: Already exists
a3224e2c3a89: Pull complete
be7a066df88f: Pull complete
bfdf741d72a9: Pull complete
ff9fc07eebd4: Pull complete
669e424fd246: Pull complete
b2d37f53b7e3: Pull complete
4d87d4b3feb8: Pull complete
d81f0fa01716: Pull complete
Digest: sha256:005e3ccd42cdb97fc609c9732708207f03729822202dab0647168e8f937e7d48
Status: Downloaded newer image for php:7.4
docker.io/library/php:7.4

Wizley:~/Project/PHP/Board # docker pull mysql:8.0.19
8.0.19: Pulling from library/mysql
619014d83c02: Pull complete
9ced578c3a5f: Pull complete
731f6e13d8ea: Pull complete
3c183de42679: Pull complete
6de69b5c2f3c: Pull complete
00f0a4086406: Pull complete
84d93aea836d: Pull complete
f18efbfd8d76: Pull complete
012b302865d1: Pull complete
fe16fd240f59: Pull complete
ca3e793e545e: Pull complete
51d0f2cb2610: Pull complete
Digest: sha256:6d0741319b6a2ae22c384a97f4bbee411b01e75f6284af0cce339fee83d7e314
Status: Downloaded newer image for mysql:8.0.19
docker.io/library/mysql:8.0.19
```

개발 환경에 사용될 각각의 버전은 다음과 같다.

```console
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               1.17                2073e0bcb60e        2 weeks ago         127MB
php                 fpm                 c17c65c110d8        2 weeks ago         405MB
php                 7.4                 7dc31b4f3403        2 weeks ago         405MB
mysql               8.0.19              791b6e40940c        2 weeks ago         465MB
```

(이 시점에서 알게 된 것은 PHP가 아닌 PHP-fpm을 사용하는 것이 좋다는 것이다. TechEmpower의 오래된 benchmark를 통해 php-fpm + php + nginx의 성능이 효율적이며 php-fpm을 통해 CGI의 처리 효율을 높일 수 있다고 한다.)

### docker compose

이 시점에서 고민을 하는 건 바로 네트워크를 물리는 작업이다. Docker 컨테이너를 frontend / backend 두 개의 네트워크로 분리한 뒤 각각 nginx+php-fpm / mysql+php-fpm 로 세팅을 하는 작업을 위해 compose를 작성할 필요가 있다.

[Networking in Compose](https://docs.docker.com/compose/networking/)

그 전에 Docker로 개발할 때 개발자들이 주로 타는 프로세스가 있다.

1. Dockerfile 작성
2. docker-compose.yml 작성
3. 개발 : docker-compose up
4. 배포 : 이미지화 및 registry, docker stack deploy

하지만 지금 시점에서 도커에 대해서 잘 모르는 상태이니 docker-compose부터 세팅을 하도록 하겠다.

도커 컴포즈를 통해서는 다음의 작업에 초점을 둔다.

1. Container의 이름
2. image 연결
3. network 분리(frontend/backend)
4. port 설정 

여러 사이트를 통해 작성한 docker-compose.yml은 다음과 같다.

```yaml
version: '3'

services:
 web:
  container_name: board_nginx
  image: nginx:1.17
  networks:
   - frontend
  ports:
   - "80:80"
  volumes:
   - ./board:/board
   - ./site.conf:/etc/nginx/conf.d/default.conf
 php:
  container_name: board_phpfpm
  image: php:fpm
  networks:
  - frontend
  - backend
  volumes:
   - ./board:/board
 db:
  container_name: board_mysql
  image: mysql:8.0.19
  networks:
   - backend
  ports:
   - "3300:3300"
  volumes:
   - ./mysql/data:/var/lib/mysql
   - ./mysql/config:/etc/mysql/conf.d
  environment:
   - MYSQL_ROOT_PASSWORD=alpine
   - MYSQL_USER=wizley
   - MYSQL_PASSWORD=alpine
networks:
 frontend:
 backend:

```

container_name은 말그대로 컨테이너의 이름을 설정하는 방법이고, image는 미리 다운받은 Docker image를 버전과 함께 명시한다. volumes의 경우 Docker내부에서 작업한 것들을 내 로컬에 저장을 하기 위해 명시한 것인데 위의 /etc/nginx/conf.d/default.conf는 도커 내부의 파일 위치이며 해당 값이 로컬의 기준으로 ./site.conf로 백업이 된다. network를 통해 frontend, backend를 명시했다.

이제 docker-compose up 명령어를 실행하면 다음과 같은 결과를 확인가능하다.

```console
Wizley:~/Project/PHP/Board # docker-compose up
Starting board_mysql    ... done
Starting board_nginx    ... done
Recreating board_phpfpm ... done
Attaching to board_mysql, board_nginx, board_phpfpm
board_mysql | 2020-02-20 04:26:57+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.19-1debian9 started.
board_mysql | 2020-02-20 04:26:57+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
board_mysql | 2020-02-20 04:26:57+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.19-1debian9 started.
php_1  | [20-Feb-2020 04:26:57] NOTICE: fpm is running, pid 1
php_1  | [20-Feb-2020 04:26:57] NOTICE: ready to handle connections
board_mysql | 2020-02-20T04:26:58.085950Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
board_mysql | 2020-02-20T04:26:58.086081Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.19) starting as process 1
board_mysql | 2020-02-20T04:26:58.484400Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
board_mysql | 2020-02-20T04:26:58.487668Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
board_mysql | 2020-02-20T04:26:58.512163Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.19'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server - GPL.
board_mysql | 2020-02-20T04:26:58.638064Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Socket: '/var/run/mysqld/mysqlx.sock' bind-address: '::' port: 33060
```

이 시점에서 docker ps -a 를 통해 세팅이 제대로 되었는지 확인해보면,

```console
Wizley:~/Project/PHP/Board # docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                                         NAMES
1f1d7824207f        php:fpm             "docker-php-entrypoi…"   About a minute ago   Up About a minute   9000/tcp                                      board_phpfpm
966990d4455e        mysql:8.0.19        "docker-entrypoint.s…"   21 minutes ago       Up About a minute   3306/tcp, 0.0.0.0:3300->3300/tcp, 33060/tcp   board_mysql
12e7e8c4f6d4        nginx:1.17          "nginx -g 'daemon of…"   52 minutes ago       Up About a minute   0.0.0.0:80->80/tcp                            board_nginx
```

정확히 compose의 설정대로 이미지가 올라간 것을 확인 가능하다.

```console
Wizley:~/Project/PHP/Board # docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
cf37099fabab        board_backend       bridge              local
55aa5bf4b62f        board_default       bridge              local
97a213511557        board_frontend      bridge              local
f7b586735a32        bridge              bridge              local
35b24f992dca        host                host                local
7138c9085194        none                null                local
```

network 명령을 통해 board_backend와 board_frontend 네트워크가 생성된 것을 확인할 수 있는데 php, mysql, nginx가 제대로 설정되었는지 까지는 확인이 불가능하였다.

```console
Wizley:~/Project/PHP/Board # docker inspect board_backend
[
    {
        "Name": "board_backend",
        "Id": "cf37099fababdaae343b9775ff847000320279bb3aff4af90d754bb09a7cdc3d",
        "Created": "2020-02-20T03:28:31.4474033Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.20.0.0/16",
                    "Gateway": "172.20.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": true,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "1f1d7824207f556488e8202b92553b1d8dc8da773c34b5d509097cea0318a591": {
                "Name": "board_phpfpm",
                "EndpointID": "a8f5c89d1c7111a4e0f839bb1115b418102b3af2e513a024cbdb9a30ee8fafd2",
                "MacAddress": "02:42:ac:14:00:03",
                "IPv4Address": "172.20.0.3/16",
                "IPv6Address": ""
            },
            "966990d4455e6ab0336d7f9133a0bc1b3f8bee66d156d5a74ad96a3cbd89f8fc": {
                "Name": "board_mysql",
                "EndpointID": "8ef4a9896952cf15457f6c1600441dabcf6854c2e4d87e06bb307f0a00ab2311",
                "MacAddress": "02:42:ac:14:00:02",
                "IPv4Address": "172.20.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {
            "com.docker.compose.network": "backend",
            "com.docker.compose.project": "board",
            "com.docker.compose.version": "1.25.4"
        }
    }
]
```

하지만 다행히도 docker inspect를 통해 board_backend의 Containers부분을 통해 board_phpfpm과 board_mysql이 같은 네트워크에 연동된 것을 확인할 수 있다. 

이제 국룰인 HelloWorld를 띄워볼 차례이다.

[Docker를 사용하여 php를 운영해보자](https://teamsmiley.github.io/2018/12/29/php-docker/)

위의 사이트를 참조하여 아래와 같이 site.conf를 작성하였다.

```console
Wizley:~/Project/PHP/Board # cat site.conf
server {
    index index.php;
    server_name wizley.com;
    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    root /board;

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```

이제 board 디렉토리로 이동하여 아래와 같이 index.html을 생성한다.

```html
Wizley:~/Project/PHP/Board/board # cat index.html
<html>
helloworld
</html>
```

그 결과 localhost에 접속하면 영롱한 helloworld 문구를 확인할 수 있다.

![helloworld](https://raw.githubusercontent.com/wizleysw/wizleysw.github.io/master/_posts/img/php_board/helloworld.png)

이제 database 설계를 진행할 것인데 가장 먼저 User 정보에 대한 부분의 설계를 해야된다.

## User DB 구현 및 계정 관련 기능 구현

이제 DB를 설계해야되는데 게시판 기능을 크게 보았을 때 2개의 부류로 나눌 수 있다. 계정과 작성글이다. 

### 계정정보 DB 설계 

회원가입 기능을 통해 계정정보를 생성할 때 가장 중요한 것은 ID와 PASSWORD일 것이다. 그리고 부가적으로 효율적인 관리를 위해서 계정 생성일자 및 고유식별번호, 계정의 정지 활성화 여부판단에 사용될 상태정보, 닉네임 정도를 들 수 있겠다. 이를 정리해서 나타내면 다음과 같다.

```
- id
- password
- created
- unique_id
- status
- nickname
```

이를 위해서 크게 User라는 데이터 베이스를 생성하고 account라는 table에 위의 정보를 추가하는 작업을 수행하도록 하겠다.

```console
Wizley:~/Project/PHP/php_board # docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                         NAMES
b734c92f7c80        nginx:1.17          "nginx -g 'daemon of…"   10 minutes ago      Up 10 minutes       0.0.0.0:80->80/tcp                            board_nginx
f68ab5883779        mysql:8.0.19        "docker-entrypoint.s…"   10 minutes ago      Up 10 minutes       3306/tcp, 0.0.0.0:3300->3300/tcp, 33060/tcp   board_mysql
205810f910ad        php:fpm             "docker-php-entrypoi…"   10 minutes ago      Up 10 minutes       9000/tcp                                      board_phpfpm

Wizley:~/Project/PHP/php_board # docker exec -it board_mysql /bin/bash
root@f68ab5883779:/# mysql -u wizley -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.19 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

exec 명령어를 통해 Container 내부로 접속하였다. 이제 database를 추가하면 된다.

```console
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
+--------------------+
1 row in set (0.01 sec)

mysql> CREATE DATABASE User;
ERROR 1044 (42000): Access denied for user 'wizley'@'%' to database 'User'
```

하지만 여기서 Access Denied를 통해 Wizley라는 계정의 권한으로 작업을 수행하지 못한다는 문구가 나온다. 이를 해결하기 위해서는 몇가지 명령어를 실행해야 한다.

```console
mysql> grant all privileges on * . * to 'wizley'@'%';
Query OK, 0 rows affected (0.01 sec)
```

위의 명령어를 수행하고 나서 다시 wizley로 로그인을 하면 다음과 같이 databases가 추가로 보이는 것을 확인 가능하다.

```console
root@f68ab5883779:/# mysql -u wizley -p
Enter password:

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

이제 User라는 데이터베이스를 추가하는 작업이 가능하다. 

```console
mysql> CREATE DATABASE User;
Query OK, 1 row affected (0.00 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| User               |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```

User라는 데이터베이스를 추가하였고 목록에 추가된 것을 확인이 가능하다. 이제 조건에 맞게 account라는 Table을 생성할 것인데 이 경우에 NOT_NULL, PRIMARY KEY등의 설정이 필요하다. uniqueID의 경우 겹치는 값이 있으면 안되기 때문에 PRIMARY KEY가 설정되어 있어야 하며, ID또는 PASSWORD또한 없으면 안되는 정보이기에 NOT_NULL 조건이 붙어야 된다는 의미이다.

```console
mysql> use User;
Database changed
mysql> CREATE TABLE Account
    -> (
    -> uniqueID INT NOT NULL AUTO_INCREMENT,
    -> userID CHAR(16) NOT NULL,
    -> password CHAR(128) NOT NULL,
    -> created DATETIME,
    -> status INT NOT NULL,
    -> nickname VARCHAR(10) NOT NULL,
    -> PRIMARY KEY(uniqueID)
    -> );
Query OK, 0 rows affected (0.02 sec)

mysql> show tables;
+----------------+
| Tables_in_User |
+----------------+
| Account        |
+----------------+
1 row in set (0.00 sec)
```

Account라는 테이블을 위의 설계와 같이 생성하였다. 이제 테스트를 위해 하나의 계정을 생성해보았다.

```console
mysql> INSERT INTO Account(userID, password, status, nickname) VALUES('admin', 'alpine', 1, 'Admin');
Query OK, 1 row affected (0.00 sec)

mysql> select * from Account;
+----------+--------+----------+---------+--------+----------+
| uniqueID | userID | password | created | status | nickname |
+----------+--------+----------+---------+--------+----------+
|        1 | admin  | alpine   | NULL    |      1 | Admin    |
+----------+--------+----------+---------+--------+----------+
1 row in set (0.00 sec)
```

### DB 연결 확인 

admin이라는 계정이 alpine이라는 패스워드로 생성된 것을 확인할 수 있다. 이제 제대로 DB가 backend 네트워크로 연결이 되었는지 확인을 하기 위해서 간단한 php 코드를 작성해볼 것이다.

```php
<?php
	$conn = mysqli_connect('db', 'wizley', 'alpine');
	if(!$conn){
		die("Connection Error!");
	}
	echo "Success!";
	mysqli_close($conn);
?>
```

위와 같은 코드를 작성하고 구동을 해보았더니 Undefined mysqli_connect 에러가 발생하였다. php-mysqli가 설치되지 않아서 발생하는 문제였다. 해당 부분을 수정하기 위해 많은 삽질을 하였지만 기존의 Container에서는 고치지 못하였기에 다른 Image를 땡겨 쓰기로 마음먹었다.

```console
FROM php:fpm-alpine
RUN docker-php-ext-install mysqli
```

위와 같이 Dockerfile을 생성하였고 build를 하였다.

```console
Wizley:~/Project/PHP/php_board # docker build --tag phpmysqli:1.0 .
Sending build context to Docker daemon  198.9MB
Step 1/2 : FROM php:fpm-alpine
fpm-alpine: Pulling from library/php
c9b1b535fdd9: Pull complete
c1c0a1817bec: Pull complete
cdd5b3ea1fc3: Pull complete
db87396003bd: Pull complete
6e71cca12e10: Pull complete
ed2310d2f791: Pull complete
601ef2217a14: Pull complete
41dc18d982f5: Pull complete
72be421f63f8: Pull complete
f10dd871243f: Pull complete
```

그 후에 설치된 image 목록을 확인해보면 phpmysqli가 추가된 것을 확인할 수 있다.

```console
Wizley:~ # docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
phpmysqli           1.0                 b5cd17f37d99        6 minutes ago       83.9MB
nginx               1.17                2073e0bcb60e        2 weeks ago         127MB
php                 fpm                 c17c65c110d8        2 weeks ago         405MB
php                 7.4                 7dc31b4f3403        2 weeks ago         405MB
mysql               8.0.19              791b6e40940c        2 weeks ago         465MB
php                 fpm-alpine          4a1ce12adee5        3 weeks ago         83.6MB
```

이제 docker-compose 파일에서도 약간의 수정이 필요하다.

```yaml
 php:
  container_name: board_phpmysqli
  image: b5cd17f37d99
  networks:
  - frontend
  - backend
  volumes:
   - ./board:/board
```

image부분에 위의 IMAGE ID를 넣어주었고 container_name도 board_phpmysqli로 변경하였다. 그 후에 구동을 하면 Success! 문구가 뜨는 것을 확인 가능하다.

이제 좀더 세부적인 데이터를 가져오도록 할 것이다. DB안에 맨 처음 추가한 admin을 조회하는 간단한 쿼리문을 짜볼 것이다.

```php
<?php
	$conn = mysqli_connect("db", "wizley", "alpine");
	if(!$conn){
		die("Connection Error!");
	}

	mysqli_select_db($conn, "User");
	$query = "SELECT * FROM Account";
	$result = mysqli_query($conn, $query);
	$row = mysqli_fetch_array($result);
	echo "UserID   : " . $row['userID'] . "<br>";
	echo "password : " . $row['password'] . "<br>";
	mysqli_close($conn);
?>
```

확인해보면 아래와 같이 admin / alpine이라는 정보를 가져온 것을 확인할 수 있다!

![queryCheck](https://raw.githubusercontent.com/wizleysw/wizleysw.github.io/master/_posts/img/php_board/queryCheck.png)

이제 원하는대로 DB의 쿼리작업이 가능하게 되었다. 

### 로그인 구현

쿼리의 조회가 가능하기 때문에 userID와 password를 사용하여 올바른 계정정보인지 검사하는 루틴에 대한 작성이 가능하다. 지금 이 단계에서는 loginForm이라는 HTML페이지를 생성한 뒤 loginCheck.php를 통해 조회하는 기능을 구현해볼 것이다.

```html
<!DOCTYPE html>
<html>
<head>
	<title>로그인</title>
	<meta charset="utf-8">
</head>
<body>
	<form action="loginCheck.php" method="POST">
		<input type="text" name="userID" placeholder="아이디"><br>
		<input type="password" name="password" placeholder="패스워드"><br>
		<button type="submit">로그인</button>
	</form>
</body>
```

위와 같이 간단하게 FORM을 생성하였고 userID와 password를 입력받은 뒤에 submit 버튼을 통해 loginCheck.php로 검사로직이 넘어가도록 구현을 하였다. 이제 loginCheck.php를 작성할 차례이다.

```php
<?php
	$conn = mysqli_connect("db", "wizley", "alpine");
	if(!$conn){
		die("Connection Error!!");
	}
	mysqli_select_db($conn, "User");
	$query = "SELECT * FROM Account WHERE userID = '{$_POST['userID']}' AND password = '{$_POST['password']}'";
	$result = mysqli_query($conn, $query);
	$row = mysqli_fetch_array($result);

	if(!$row){
		echo '<script>alert("아이디 또는 패스워드가 올바르지 않습니다.");';
		echo 'location.href="/loginForm.php";</script>';
		exit;
	}

	echo $row['userID'] . "님";
	mysqli_close($conn);
?>
```

POST로 전송된 userID와 password가 db의 User Database의 Account 테이블 내에서 올바른 값인지 조회된 뒤 결과에 따라 userID를 출력하거나 loginForm으로 리다이렉트를 한다. 

![loginForm](https://raw.githubusercontent.com/wizleysw/wizleysw.github.io/master/_posts/img/php_board/loginForm.png)

### 로그아웃 구현 

로그인이 있으면 로그아웃 기능도 구현이 되어야 한다. 하지만 대부분의 유저들이 로그아웃을 진행하지 않기 때문에 Cookie를 생성하여 expire date를 구현하는게 좋아보인다. 이 시점에서 Session과 Cookie에 대한 개념을 아는 것이 중요한데 가장 큰 차이는 세션은 정보를 관리하는 대상이 서버인지 클라이언트인지가 다르다는 점이다. 

[쿠키와 세션 개념](https://interconnection.tistory.com/74)

쿠키를 통해 로그인 유지 상태에 변경을 주기 위해서는 아이디와 패스워드에 대한 검증이 끝난 직후에 발급을 하는 것이 가장 좋을 것이다. loginCheck.php를 다음과 같이 수정한다. 

```php
<?php
	$conn = mysqli_connect("db", "wizley", "alpine");
	if(!$conn){
		die("Connection Error!!");
	}
	mysqli_select_db($conn, "User");
	$query = "SELECT * FROM Account WHERE userID = '{$_POST['userID']}' AND password = '{$_POST['password']}'";
	$result = mysqli_query($conn, $query);
	$row = mysqli_fetch_array($result);

	if(!$row){
		echo '<script>alert("아이디 또는 패스워드가 올바르지 않습니다.");';
		echo 'location.href="/loginForm.php";</script>';
		exit;
	}
	setcookie("expireTime", $_POST['userID'], time()+3600);
	echo '<script>location.href="/index.php"</script>';
	
	mysqli_close($conn);
?>
```

이제 로그인에 성공하면 expireTime이라는 이름으로 userID 값을 가진 쿠키가 1시간동안 유지되게 된다. 이제 로그인을 한 후에 이동할 페이지를 만들어야 되는데 이름은 index.php로 지을 것이다.

```php
<?php
	if(isset($_COOKIE['expireTime'])){
		echo "로그인 정보 " . $_COOKIE['expireTime'];
	}

	else{
		echo '<script>alert("로그인 페이지로 이동합니다.");';
		echo 'location.href="/loginForm.php";</script>';
	}
?>
```

COOKIE에서 expireTime을 가져와 존재할 경우에는 그 값인 userID를 적어주고 아닌 경우에는 loginForm으로 이동시키는 간단한 로직이다.

이제 로그아웃을 만들차례인데, 먼저 COOKIE를 확인하여 로그인한 상태인지를 본다. 그 후 로그아웃 버튼을 누르면 로그아웃이 진행되도록 할 것이다.

```php
<?php
	if(isset($_COOKIE['expireTime'])){
		echo '로그인 정보 ' . $_COOKIE['expireTime'] . '<br>';
		echo '<a href="/logout.php">로그아웃</a>';
	}

	else{
		echo '<script>alert("로그인 페이지로 이동합니다.");';
		echo 'location.href="/loginForm.php";</script>';
	}
?>
```

다시 index.php에 버튼을 하나 추가해준다. 이제 이 버튼을 누르면 쿠키의 만료 기간을 현재의 시간보다 이전으로 변경함으로써 효력을 다하도록 만든다.

```php
<?php
	setcookie("expireTime", "", time()-99999999);
?>
<script type="text/javascript">
	location.href="/index.php";
</script>
```

이제 로그아웃 버튼을 클릭하면 로그아웃이 진행된다!

### 회원가입 구현

로그인 부분이 대략적으로 구현되었으니 이제 회원가입 기능의 구현이 필요하다. 이를 위해서 loginForm 부분에 회원가입 링크를 추가해준다.

```html
<!DOCTYPE html>
<html>
<head>
  <title>로그인</title>
  <meta charset="utf-8">
</head>
<body>
  <form action="loginCheck.php" method="POST">
    <input type="text" name="userID" placeholder="아이디"><br>
    <input type="password" name="password" placeholder="패스워드"><br>
    <button type="submit">로그인</button>
    <a href="signUp.php">회원가입</a>
  </form>
</body>
```

그리고 회원가입을 처리하기 위한 signUp.php를 생성한다. 여기서는 INSERT를 통해 계정정보를 USER database의 Account에 넣는 작업을 수행할 것이다. 

```php
<?php
  $conn = mysqli_connect('db', 'wizley', 'alpine');
  if(!$conn){
    die("Connection Error!");
  }

  $dateNow = date("Y-m-d H:i:s");
  mysqli_select_db($conn, "User");
  $query = "
    INSERT INTO Account(userID, password, nickname, created, status) 
    VALUES('{$_POST['userID']}', '{$_POST['password']}', '{$_POST['nickname']}', '$dateNow', 1)
  ";

  $result = mysqli_query($conn, $query);
  if(!$result){
    echo '<script>alert("정보를 다시 확인해주시기 바랍니다.");';
    echo 'history.back();</script>';
    exit;
  }
?>

<script type="text/javascript">
  alert("회원가입이 완료되었습니다.");
  location.href="/loginForm.php";
</script>
```

테스트를 수행하기 전에 TABLE의 정보를 보면 다음과 같다.

```console
mysql> select * from Account;
+----------+--------+----------+---------+--------+----------+
| uniqueID | userID | password | created | status | nickname |
+----------+--------+----------+---------+--------+----------+
|        1 | admin  | alpine   | NULL    |      1 | Admin    |
+----------+--------+----------+---------+--------+----------+
1 row in set (0.00 sec)
```

이제 abcd / abcd / abcd로 아이디, 패스워드, 닉네임을 지정하고 회원가입을 진행하면 다음의 상태가 된다.

```console
mysql> select * from Account;
+----------+--------+----------+---------------------+--------+----------+
| uniqueID | userID | password | created             | status | nickname |
+----------+--------+----------+---------------------+--------+----------+
|        1 | admin  | alpine   | NULL                |      1 | Admin    |
|        4 | abcd   | abcd     | 2020-02-20 10:14:44 |      1 | abcd     |
+----------+--------+----------+---------------------+--------+----------+
2 rows in set (0.00 sec)
```

데이터 값은 잘 들어간 것을 확인할 수 있다. 여기서 uniqueID가 4인 이유는 2,3을 테스트에 사용하였기 때문이다. 이제 보안의 관점에서 봤을때 3가지 큰 로직 문제를 들 수 있다. 

1. userId와 nickname의 중복검사 수행여부
2. password 필드의 평문 저장된 패스워드 정보
3. 아무값도 입력하지 않아도 회원가입됨

물론 여기서 xss라던가 sqli 취약점이 존재하겠지만 그 둘은 배제하고 로직적으로 구현이 필요한 부분은 위와 같이 요약할 수 있겠다. 그러면 중복검사 부분부터 추가하는 작업을 수행해보자. INSERT부분이 수행되기 전에 해당 userID랑 nickname을 조회하여 존재하는지에 대한 예외처리를 진행하면 된다.

```php
  mysqli_select_db($conn, "User");
  $query = "SELECT * FROM Account WHERE userID = '{$_POST['userID']}'";
  $result = mysqli_query($conn, $query);
  if(mysqli_num_rows($result)>0){
    echo '<script>alert("이미 존재하는 아이디입니다.");';
    echo 'history.back();</script>';
    exit;
  }

  $query = "SELECT * FROM Account WHERE nickname = '{$_POST['nickname']}'";
  $result = mysqli_query($conn, $query);
  if(mysqli_num_rows($result)>0){
    echo '<script>alert("다른 닉네임을 선택해주세요.");';
    echo 'history.back();</script>';
    exit;
  }

  $dateNow = date("Y-m-d H:i:s");
  $query = "
    INSERT INTO Account(userID, password, nickname, created, status) 
    VALUES('{$_POST['userID']}', '{$_POST['password']}', '{$_POST['nickname']}', '$dateNow', 1)
  ";

  $result = mysqli_query($conn, $query);
  if(!$result){
    echo '<script>alert("정보를 다시 확인해주시기 바랍니다.");';
    echo 'history.back();</script>';
    exit;
  }
```

조회를 위해서 Account에서 userID와 nickname으로 쿼리를 조회해 한개라도 존재하면 history.back();을 하도록 세팅하였다. 이 과정에서 mysqli_num_rows($result)>0 을 mysqli_num_rows($result>0) 으로 잘못 작성하는 바람에 에러가 발생하지 않고 회원가입이 진행하였고 디비는 다음과 같이 더럽혀졌다. 

```console
mysql> select * from Account;
+----------+--------+----------+---------------------+--------+----------+
| uniqueID | userID | password | created             | status | nickname |
+----------+--------+----------+---------------------+--------+----------+
|        1 | admin  | alpine   | NULL                |      1 | Admin    |
|        4 | abcd   | abcd     | 2020-02-20 10:14:44 |      1 | abcd     |
|        5 | abcd   | 1        | 2020-02-20 10:21:29 |      1 | 1        |
|        6 | admin  | 1        | 2020-02-20 10:22:17 |      1 | 1        |
|        7 | abcd   | 1        | 2020-02-20 10:23:08 |      1 | 1        |
|        8 | 11     | 1        | 2020-02-20 10:23:24 |      1 | 1        |
|        9 | admin  | 1        | 2020-02-20 10:23:32 |      1 | 1        |
|       10 | 123    | ????     | 2020-02-20 10:23:45 |      1 | 123      |
|       11 | admin  | a        | 2020-02-20 10:24:28 |      1 |          |
|       12 |        |          | 2020-02-20 10:28:35 |      1 |          |
+----------+--------+----------+---------------------+--------+----------+
```

쿼리에 대한 로깅을 하기 위해서 아래와 같이 설정을 변경하였다.

```console
mysql> SHOW VARIABLES LIKE '%general%';
+------------------+---------------------------------+
| Variable_name    | Value                           |
+------------------+---------------------------------+
| general_log      | OFF                             |
| general_log_file | /var/lib/mysql/f68ab5883779.log |
+------------------+---------------------------------+
2 rows in set (0.01 sec)

mysql>
mysql> SET GLOBAL general_log = ON;
Query OK, 0 rows affected (0.02 sec)

mysql> SHOW VARIABLES LIKE '%general%';
+------------------+---------------------------------+
| Variable_name    | Value                           |
+------------------+---------------------------------+
| general_log      | ON                              |
| general_log_file | /var/lib/mysql/f68ab5883779.log |
+------------------+---------------------------------+
2 rows in set (0.01 sec)
```

그리고 해당 경로의 log파일을 확인해보면 다음과 같이 쿼리를 확인할 수 있다.

```console
root@f68ab5883779:/# cat /var/lib/mysql/f68ab5883779.log
/usr/sbin/mysqld, Version: 8.0.19 (MySQL Community Server - GPL). started with:
Tcp port: 3306  Unix socket: /var/run/mysqld/mysqld.sock
Time                 Id Command    Argument
2020-02-20T10:34:42.900555Z    29 Query SHOW VARIABLES LIKE '%general%'
2020-02-20T10:35:40.703799Z    86 Connect wizley@board_phpmysqli.php_board_backend on  using TCP/IP
2020-02-20T10:35:40.704496Z    86 Init DB User
2020-02-20T10:35:40.705589Z    86 Query SELECT * FROM Account WHERE userID = 'admin'
2020-02-20T10:35:40.706479Z    86 Query SELECT * FROM Account WHERE nickname = '123'
2020-02-20T10:35:40.707205Z    86 Query INSERT INTO Account(userID, password, nickname, created, status)
    VALUES('admin', '123', '123', '2020-02-20 10:35:40', 1)
2020-02-20T10:35:40.709930Z    86 Quit
```

여기서 내가 넣은 쿼리가 제대로 작동을 했기에 멘붕이 왔지만 스트레스받으면서 소스코드를 계속 확인하다보니 발견할 수 있었다. 이제 저 부분을 통해서 중복에 대한 검사가 진행된다. 

이제 두 번째로 고쳐야되는 부분은 password의 평문저장이다. DB를 설계할 때 hash화를 고려했기 때문에 해당 필드의 길이를 길게 두었었다. 

[PHP : password_hash 함수로 암호화](https://m.blog.naver.com/psj9102/221223524085)

위의 링크를 확인해보면 password_hash라는 함수가 php 내부에 구현되어 있는 것을 확인할 수 있다. 이제 해당 부분을 INSERT 문에 적용해주면 된다.

```php
  $dateNow = date("Y-m-d H:i:s");
  $passwordHashed = password_hash($_POST['password'], PASSWORD_DEFAULT);
  $query = "
    INSERT INTO Account(userID, password, nickname, created, status) 
    VALUES('{$_POST['userID']}', '$passwordHashed', '{$_POST['nickname']}', '$dateNow', 1)
  ";
```

이제 이 값이 제대로 적용이 되서 login이 가능한지를 확인하기 위해서는 login쪽의 로직을 변경해야 된다.

```php
<?php
  $conn = mysqli_connect("db", "wizley", "alpine");
  if(!$conn){
    die("Connection Error!!");
  }
  mysqli_select_db($conn, "User");
  $query = "SELECT * FROM Account WHERE userID = '{$_POST['userID']}'";
  $result = mysqli_query($conn, $query);
  $row = mysqli_fetch_array($result);

  if(!$row){
    echo '<script>alert("아이디 또는 패스워드가 올바르지 않습니다.");';
    echo 'location.href="/loginForm.php";</script>';
    exit;
  }

  if(password_verify($_POST['password'], $row['password'])){
    setcookie("expireTime", $_POST['userID'], time()+3600);
    echo '<script>location.href="/index.php"</script>';
  }

  else{
    echo '<script>alert("아이디 또는 패스워드가 올바르지 않습니다.");';
    echo 'location.href="/loginForm.php";</script>';
    exit;
  }

  mysqli_close($conn);
?>
```

loginCheck 부분을 다음과 같이 변경하였다. 그리고 abcd라는 아이디에 대해서 로그인을 시도하면 로그인에 성공하는 것을 확인 가능하다! 이제 이 과정에서 get_magic_quotes_gpc() 등을 통해 좀더 POST로 넘어오는 데이터에 대한 검증을 강화하도록 하겠다.

[회원가입 폼 검증 후 출력](https://idkwim.tistory.com/154)

위의 사이트의 유용한 부분을 가져와서 추가하였다.

```php
<?php
  $conn = mysqli_connect("db", "wizley", "alpine");
  if(!$conn){
    die("Connection Error!!");
  }

  function fix_string($string){
    if(get_magic_quotes_gpc()) $string=stripslashes($string);
    return htmlentities($string);
  }

  $id=$pw="";

  if(isset($_POST['userID']))
    $id=fix_string($_POST['userID']);
  if(isset($_POST['password']))
    $pw=fix_string($_POST['password']);

  if(strlen($id)<4){
    echo '<script>alert("아이디를 잘못 입력하셨습니다.");';
    echo 'location.href="/loginForm.php";</script>';
    exit;
  }

  if(strlen($pw)<4){
    echo '<script>alert("패스워드를 다시 입력해주세요.");';
    echo 'location.href="/loginForm.php";</script>';
    exit;
  }

  mysqli_select_db($conn, "User");
  $query = "SELECT * FROM Account WHERE userID = '$id'";
  $result = mysqli_query($conn, $query);
  $row = mysqli_fetch_array($result);

  if(!$row){
    echo '<script>alert("아이디 또는 패스워드가 올바르지 않습니다.");';
    echo 'location.href="/loginForm.php";</script>';
    exit;
  }

  if(password_verify($pw, $row['password'])){
    setcookie("expireTime", $id, time()+3600);
    echo '<script>location.href="/index.php"</script>';
  }

  else{
    echo '<script>alert("아이디 또는 패스워드가 올바르지 않습니다.");';
    echo 'location.href="/loginForm.php";</script>';
    exit;
  }

  mysqli_close($conn);
?>
```

### prepared statement

조금 보완되긴 했지만 sqli를 막기 위해서는 prepared statement를 사용하는게 좋을 것 같다. 바꿔보도록 하자.

```php
$conn = new mysqli("db", "wizley", "alpine", "User");
if(!$conn){
  die("Connection Error!!");
}

$query = "SELECT* FROM Account WHERE userID LIKE ?";
$stmt = $conn->stmt_init();
$stmt = $conn->prepare($query);
$stmt->bind_param("s", $id);
$stmt->execute();
$result = $stmt->get_result();
$row = mysqli_fetch_array($result);
```

[prepared statement 예제](http://www.fun25.co.kr/blog/php-mysqli-simple-sample)

위와 같이 약간의 변경을 하였고, prepare와 bind_param을 통해 파라미터의 데이터 타입등에 대한 설정을 하였다. 

```console
2020-02-20T13:41:37.437438Z   135 Prepare INSERT INTO Account(userID, password, nickname, created, status)
    VALUES(?,?,?,'2020-02-20 13:41:37',1)
2020-02-20T13:41:37.437713Z   135 Execute INSERT INTO Account(userID, password, nickname, created, status)
    VALUES('test3','$2y$10$Z5KBtyAqTkrwZ.cp7GMW3OQlOdflgyHl8bTFwJ9VnD9.8zA3/fPBe','test3','2020-02-20 13:41:37',1)
2020-02-20T13:41:37.439564Z   135 Close stmt
```

정상적으로 작동하는 것까지 확인이 된다. 

### Session

이제 세션을 추가할 차례이다. PHP에서는 session_start()를 통해 세션값을 생성한다고 한다. 마찬가지로 로그인이 되는 그 시점에 발급하는 것이 가장 좋아보인다. 해당 값은 기존의 COOKIE를 대체하여 로그인 여부를 확인하는 flag의 역할로 활용될 것이다.

```php
if(password_verify($pw, $row['password'])){
  session_start();
  $_SESSION['USERSESSION'] = $id;
  setcookie("expireTime", $id, time()+3600);
  echo '<script>location.href="/index.php"</script>';
}
```

loginCheck.php 부분에서 SESSION을 추가해준다. 그리고 다른 php파일들에는 session_start()를 추가해주면 된다. 

```php
<?php
  session_start();
  if(isset($_SESSION['USERSESSION'])){
  echo '로그인 정보 ' . $_SESSION['USERSESSION'] . '<br>';
  echo '<a href="/logout.php">로그아웃</a>';
  }

  else{
    echo '<script>alert("로그인 페이지로 이동합니다.");';
    echo 'location.href="/loginForm.php";</script>';
  }
?>
```

이제 로그인 후에 SESSION에 대한 정보를 확인해보면 아래와 같이 생성된 것을 확인이 가능하다.

![session](https://raw.githubusercontent.com/wizleysw/wizleysw.github.io/master/_posts/img/php_board/session.png)

그리고 서버쪽의 /tmp 폴더의 내부에 세션파일이 저장된다.

```console
/tmp # ls -al
total 16
drwxrwxrwt    1 root     root          4096 Feb 21 10:32 .
drwxr-xr-x    1 root     root          4096 Feb 20 07:04 ..
-rw-------    1 www-data www-data         0 Feb 20 13:54 sess_0f3199d2a07722100a88c9a6f94408b9
-rw-------    1 www-data www-data         0 Feb 20 13:55 sess_46065d93eaa6a2970aa186fd18ef4654
-rw-------    1 www-data www-data        23 Feb 20 13:58 sess_6595d62b49e2689cb2554439fca4e43a
-rw-------    1 www-data www-data         0 Feb 20 13:54 sess_79e56a43f802556787697272a23f3dbf
-rw-------    1 www-data www-data        23 Feb 21 10:44 sess_c99d3d8bbe51b9787f8896c84e851b73
/tmp # cat sess_c99d3d8bbe51b9787f8896c84e851b73
USERSESSION|s:4:"abcd";/tmp #

```

해당 값을 지우면 로그인이 되어 있지 않은 상태로 판단하여 loginForm.php을 띄우게 된다. logout 부분에서는 세션에 대한 정보를 제거해야하기 때문에 아래와 같이 session_destroy(); 를 추가해주면 된다.

```php
<?php
  session_start();
  setcookie("expireTime", "", time()-99999999);
  session_destroy();
?>
```

여기까지 회원가입과 로그인과 관련된 간단한 골격에 대한 설계가 마무리 되었다. 

### 소스코드 모듈화

새로운 DB를 설계하기 전에 약간의 코드 리팩토링을 진행하도록 할건데 겹치는 기능을 수행하는 코드를 하나의 독립된 php파일로 분리하는 작업이다.

```php
<!DOCTYPE html>
<html>
<head>
  <title><?php echo $title; ?></title>
</head>
```

HTML파일의 윗 부분은 매번 같은 구조로 설계를 진행하고 있으니 그 타이틀의 값만 변경하면 된다. 그리고 이에 맞춰서 loginForm.php를 다음과 같이 변경하면 된다.

```html
<?php
  $title = "로그인";
  require_once('head.php');
?>
<meta charset="utf-8">
<body>
  <form action="loginCheck.php" method="POST">
    <input type="text" name="userID" placeholder="아이디" required><br>
    <input type="password" name="password" placeholder="패스워드" required><br>
    <button type="submit">로그인</button>
    <a href="signUp.php">회원가입</a>
  </form>
</body>
```

DB부분도 mysqli를 요청하는 부분은 동일하기 때문에 분리를 진행한다.

```php
<?php
  $conn = new mysqli("db", "wizley", "alpine", "User");
  if(!$conn){
    die("Connection Error!");
  }
?>
```

UserDBconnect.php라는 독립된 파일로 분리함으로써 db명 또는 Database의 이름을 한 파일에서만 변경해주면 같은 DB를 사용하고 있는 여러 php에 대한 값을 한 번에 변경가능하다. 이 시점에서 또 드는 고민이 게시판 기능의 작성자명으로 nickname을 사용하는 것이 낫다는 것이다. 그리고 그 값은 session에 저장되면 좋을 것 같아서 로직 수정을 하도록 하겠다.

```php
  $query = "SELECT* FROM Account WHERE userID LIKE ?";
  $stmt = $conn->stmt_init();
  $stmt = $conn->prepare($query);
  $stmt->bind_param("s", $id);
  $stmt->execute();
  $result = $stmt->get_result();
  $row = mysqli_fetch_array($result);

  if(!$row){
    echo '<script>alert("아이디 또는 패스워드가 올바르지 않습니다.");';
    echo 'location.href="/loginForm.php";</script>';
    exit;
  }

  if(password_verify($pw, $row['password'])){
    session_start();
    $_SESSION['USERSESSION'] = $id;
    $_SESSION['NICKNAME'] = $row['nickname'];
    setcookie("expireTime", $id, time()+3600);
    echo '<script>location.href="/index.php"</script>';
  }
```

다행히 기존의 result값을 row에 저장해놓았기에 해당 값을 기준으로 NICKNAME이라는 값을 하나 더 저장해놓으면 됬다. 그리고 다시 서버의 세션상태를 보면 다음과 같이 닉네임이 저장되어 있는 것을 확인 가능하다.

```console
/tmp # ls -al
total 16
drwxrwxrwt    1 root     root          4096 Feb 21 10:48 .
drwxr-xr-x    1 root     root          4096 Feb 20 07:04 ..
-rw-------    1 www-data www-data         0 Feb 20 13:54 sess_0f3199d2a07722100a88c9a6f94408b9
-rw-------    1 www-data www-data         0 Feb 20 13:55 sess_46065d93eaa6a2970aa186fd18ef4654
-rw-------    1 www-data www-data        23 Feb 20 13:58 sess_6595d62b49e2689cb2554439fca4e43a
-rw-------    1 www-data www-data         0 Feb 20 13:54 sess_79e56a43f802556787697272a23f3dbf
-rw-------    1 www-data www-data        42 Feb 21 10:48 sess_c99d3d8bbe51b9787f8896c84e851b73
/tmp # cat sess_c99d3d8bbe51b9787f8896c84e851b73
```

## Board DB 구현 및 게시판 기능 구현

이제 게시판 부분의 구현을 진행할 차례이다. User와 마찬가지로 새로운 DB를 생성하여 Board라는 이름을 지어줄 것이다. 그리고 Table 명으로 FreeBoard를 주어서 간단한 자유게시판의 기능을 구현하는데 까지 진행한 뒤에 이 프로젝트를 마무리할 예정이다. 물론 실제 게시판 배포에서는 좋아요/싫어요, 댓글 기능 등 추가적인 구현이 필요하겠지만 PHP는 이런식으로 돌아가는구나를 아는 정도로 마칠 프로젝트라 Django를 구현하는 과정에서 고민을 해보도록 하겠다..


### 게시판 DB 설계

게시판의 주요 컨텐츠는 다음과 같이 요약할 수 있다.

1.글 번호 
2.글 제목
3.글 작성자
4.글 작성시간
5.글 내용
6.비밀글 여부 
7.비밀글 패스워드
8.전체/로그인 글 여부
9.조회수 

여기서 글 번호는 고유 번호로 PRIMARY KEY를 사용하여 구현하면 될 것 같고 작성자는 세션을 통해 얻어오는 것이 가능하다. 전체/로그인의 경우 게시판 글이 권한을 구분해주는 것인데 1일 경우 로그인을 하지 않은 계정으로도 열람이 가능하도록 하며, 그 외에는 로그인을 해야지만 글의 컨텐츠를 확인할 수 있도록 할 것이다. 또한 만약의 경우를 생각해서 2의 값으로는 게시판을 블락하는 용도로도 발전할 수 있을 것 같다. 비밀글은 글에 패스워드를 걸어서 특정 사용자만 볼 수 있도록 하는 것이다. 

이제 DB를 설계해보자.

```console
mysql> CREATE DATABASE Board;
Query OK, 1 row affected (0.01 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| Board              |
| User               |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
6 rows in set (0.00 sec)
```

정상적으로 Board라는 database를 생성하였다. 이제 해당 DB아래에 FreeBoard 테이블을 생성할 것이다. 

```console
mysql> use Board;
Database changed
mysql> CREATE TABLE FreeBoard
    -> (
    -> boardNo int NOT NULL AUTO_INCREMENT,
    -> title VARCHAR(16) NOT NULL,
    -> author VARCHAR(16) NOT NULL,
    -> time datetime NOT NULL,
    -> content text NOT NULL,
    -> secret int,
    -> secretPassword VARCHAR(128),
    -> permission int NOT NULL,
    -> PRIMARY KEY(boardNo)
    -> )
    -> ;
Query OK, 0 rows affected (0.02 sec)
```

자 이제 DB에 대한 설계를 완료하였다. 이제 게시판 기능을 구현하면 된다.

### 게시판 글 목록 조회 구현

가장 먼저 만들 페이지는 글 목록을 읽어와서 뿌려주는 페이지이다. 이를 위해서는 Board DB의 FreeBoard 테이블에 쿼리를 요청하여 response를 for문으로 받아와서 뿌려주어야 한다.

```php
<?php
  session_start();
  $title = "자유게시판";
  require_once('head.php');
  if(isset($_SESSION['USERSESSION'])){
  echo '로그인 정보 ' . $_SESSION['NICKNAME'] . '<br>';
  echo '<a href="/logout.php">로그아웃</a>';
  }
?>
<meta charset="utf-8">

<body>
<div id="board">
  <table>
    <tr>
      <th>번호</th>
      <th>제목</th>
      <th>열람여부</th>
      <th>닉네임</th>
      <th>작성일</th>
    </tr>

    <?php
      require_once('BoardDBConnect.php');
      $count = 0;
      $query = "SELECT* FROM FreeBoard";
      $stmt = $conn->stmt_init();
      $stmt = $conn->prepare($query);
      $stmt->execute();
      $result = $stmt->get_result();
      while($row = mysqli_fetch_array($result)){
        $count += 1;
        echo "<tr>";
        echo "<td>{$count}</td>";
        echo "<td>{$row['title']}</td>";
        if($row['secret']==1){
          echo "<td>비밀글</td>";
        }
        else if($row['permission']==1){
          echo "<td>전체공개</td>";
        }
        else{
          echo "<td>회원전용</td>";
        }
        echo "<td>{$row['author']}</td>";
        echo "<td>{$row['time']}</td>";
      }
      $stmt->close();
      $conn->close();
    ?>
  </table>
</div>
</body>
</html>
```

prepared_statement를 활용하여 board.php를 생성하였다. 그 과정에서 DB에 대한 모듈화 코드 또한 추가하였다. 

```php
<?php
  $conn = new mysqli("db", "wizley", "alpine", "Board");
  if(!$conn){
    die("Connection Error!");
  }
?>
```

이제 정상적으로 조회하여 값을 가져올 수 있는지를 확인할 차례이다.

```console
mysql> INSERT INTO FreeBoard(title, author, time, content, secret, permission) VALUES('HelloWorld', 'Admin', '2020-02-20', 'ABCDEF', '0', '1');
Query OK, 1 row affected (0.00 sec)

mysql> select * from FreeBoard;
+---------+------------+--------+---------------------+---------+--------+----------------+------------+
| boardNo | title      | author | time                | content | secret | secretPassword | permission |
+---------+------------+--------+---------------------+---------+--------+----------------+------------+
|       1 | HelloWorld | Admin  | 2020-02-20 00:00:00 | ABCDEF  |      0 | NULL           |          1 |
+---------+------------+--------+---------------------+---------+--------+----------------+------------+
1 row in set (0.00 sec)

mysql> INSERT INTO FreeBoard(title, author, time, content, secret, permission) VALUES('HelloWorld2', 'Admin', '2020-02-20', 'ABCDEF', '0', '1');
Query OK, 1 row affected (0.00 sec)

mysql> select * from FreeBoard;                                                                                                     +---------+-------------+--------+---------------------+---------+--------+----------------+------------+
| boardNo | title       | author | time                | content | secret | secretPassword | permission |
+---------+-------------+--------+---------------------+---------+--------+----------------+------------+
|       1 | HelloWorld  | Admin  | 2020-02-20 00:00:00 | ABCDEF  |      0 | NULL           |          1 |
|       2 | HelloWorld2 | Admin  | 2020-02-20 00:00:00 | ABCDEF  |      0 | NULL           |          1 |
+---------+-------------+--------+---------------------+---------+--------+----------------+------------+
2 rows in set (0.00 sec)

```

테스트를 위해서 임의로 글 데이터 2개를 넣어주었다. 이제 board.php에 접속하여 확인을 해보면 다음과 같은 결과를 확인할 수 있다. 


![board](https://raw.githubusercontent.com/wizleysw/wizleysw.github.io/master/_posts/img/php_board/board.png)

### 게시판 글 내용 조회 구현

이제 게시판의 목록을 토대로 세부 내용을 조회하는 루틴의 구현이 필요하다. 여기서 고려해야될 사항은 다음과 같다.

1. 게시판 번호에 대한 검증
2. 게시판 권한 검증
3. 게시판 비밀글 여부 확인

1번같은 경우 no는 PRIMARY_KEY로 1부터 증가한다. 하지만 GET으로 넘어오는 파라미터의 값에 대한 검증이 없다면 no=0과 같이 존재하지 않는 게시물에 대한 조회요청이 들어올 것이다. 두 번째로는 게시판 권한 검증인데 로그인한 유저만 확인이 가능한 경우에는 SESSION 검증이 필요하다. 비밀글 같은 경우에는 당연히 패스워드를 입력받는 루틴이 추가되어야 한다.

```php
<?php
  session_start();
  $title = "글 보기";
  require_once('head.php');
  if(isset($_SESSION['USERSESSION'])){
  echo '로그인 정보 ' . $_SESSION['NICKNAME'] . '<br>';
  echo '<a href="/logout.php">로그아웃</a>';
  }
?>
<meta charset="utf-8">

<body>
<div id="view_board">

  <?php
    function fix_string($string){
      if(get_magic_quotes_gpc()) $string=stripslashes($string);
      return htmlentities($string);
    }

    $no="";
    if(isset($_GET['no']))
      $no=fix_string($_GET['no']);

    if($no<=0 or !$no){
      echo '<script>alert("잘못된 접근입니다!");';
      echo 'history.back();</script>';
      exit;
    }

    require_once('BoardDBConnect.php');
    $query = "SELECT * FROM FreeBoard WHERE boardNo LIKE ?";
    $stmt = $conn->stmt_init();
    $stmt = $conn->prepare($query);
    $stmt->bind_param("i", $no);
    $stmt->execute();
    $result = $stmt->get_result();
    $row = mysqli_fetch_array($result);

    if(!$row){
      echo '<script>alert("잘못된 접근입니다!");';
      echo 'history.back();</script>';
      exit;
    }

    if($row['permission'] == 2){
      if(!isset($_SESSION['USERSESSION'])){
        echo '<script>alert("로그인이 필요합니다.");';
        echo 'location.href="/loginForm.php";</script>';
        exit;
      }
    }

    else if($row['permission'] == 3){
      echo '<script>alert("제한된 게시물입니다.");';
      echo 'history.back();</script>';
      exit;
    }

    if($row['secret'] == 1){
      echo '<script>alert("비밀글입니다.");';
      echo 'location.href="/secretBoard.php";</script>';
      exit;
    }

    echo "<table>";
    echo "<tr>";
    echo "<th>번호</th>";
    echo "<td>{$no}</td>";
    echo "</tr>";
    echo "<tr>";
    echo "<th>제목</th>";
    echo "<td>{$row['title']}</td>";
    echo "</tr>";
    echo "<tr>";
    echo "<th>열람여부</th>";
    if($row['secret']==1){
      echo "<td>비밀글</td>";
    }
    else if($row['permission']==1){
      echo "<td>전체공개</td>";
    }
    else{
      echo "<td>회원전용</td>";
    }
    echo "</tr>";
    echo "<tr>";
    echo "<th>닉네임</th>";
    echo "<td>{$row['author']}</td>";
    echo "</tr>";
    echo "<tr>";
    echo "<th>작성일</th>";
    echo "<td>{$row['time']}</td>";
    echo "</tr>";
    echo "<th>내용</th>";
    echo "<td>{$row['content']}</td>";
    echo "</table>";

    $stmt->close();
    $conn->close();
  ?>
</div>
<button type="button" onclick="location.href='javascript:history.back();'">뒤로가</button>
</body>
</html>

```

파라미터는 view.php?no=1과 같은 형식으로 넘어오게 된다. 맨처음 no에 대한 검증을 진행하고 쿼리를 조회하여 권한에 대한 부분을 확인한다. 그 후 권한에 맞게 조건문을 탄 뒤, 조건 값에 맞는 경우 결과를 출력해준다. 결과를 확인해보면 다음과 같이 조회가 되는 것을 확인가능하다.


![view](https://raw.githubusercontent.com/wizleysw/wizleysw.github.io/master/_posts/img/php_board/view.png)


이제 view 부분에서 제목을 클릭하면 해당 게시물로 이동하도록 설계를 마무리하면 된다.

```php
<div id="board">
  <table>
    <tr>
      <th>번호</th>
      <th>제목</th>
      <th>열람여부</th>
      <th>닉네임</th>
      <th>작성일</th>
    </tr>

    <?php
      require_once('BoardDBConnect.php');
      $count = 0;
      $query = "SELECT* FROM FreeBoard";
      $stmt = $conn->stmt_init();
      $stmt = $conn->prepare($query);
      $stmt->execute();
      $result = $stmt->get_result();
      while($row = mysqli_fetch_array($result)){
        $count += 1;
        echo "<tr>";
        echo "<td>{$count}</td>";
        echo "<td><a href='/view.php?no={$row['boardNo']}'>{$row['title']}</a></td>";
        if($row['secret']==1){
          echo "<td>비밀글</td>";
        }
        else if($row['permission']==1){
          echo "<td>전체공개</td>";
        }
        else{
          echo "<td>회원전용</td>";
        }
        echo "<td>{$row['author']}</td>";
        echo "<td>{$row['time']}</td>";
      }
      $stmt->close();
      $conn->close();
    ?>
  </table>
</div>
```

이제 비밀글 부분을 해결할 차례이다. 기존 게시판과는 또 다르게 비밀글에 대한 패스워드를 입력하는 루틴이 추가적으로 필요하다. 

```php
<?php
  session_start();
  $title = "글 보기";
  require_once('head.php');
  if(isset($_SESSION['USERSESSION'])){
  echo '로그인 정보 ' . $_SESSION['NICKNAME'] . '<br>';
  echo '<a href="/logout.php">로그아웃</a>';
  }
?>
<meta charset="utf-8">
<body>
<?php
    function fix_string($string){
      if(get_magic_quotes_gpc()) $string=stripslashes($string);
      return htmlentities($string);
    }

    $no="";
    if(isset($_GET['no']))
      $no=fix_string($_GET['no']);

    if($no<=0 or !$no){
      echo '<script>alert("잘못된 접근입니다!");';
      echo 'history.back();</script>';
      exit;
    }
?>
  <form action="/view.php" method="POST">
    <input type="hidden" id="no" name="no" value="<?php echo $no?>">
    <input type="secretPassword" id="secretPassword" name="secretPassword" placeholder="패스워드" required><br>
    <button type="submit">확인</button>
    <a href="/board.php">돌아가기</a>
  </form>
</body>
</html>
```

패스워드를 입력받는 secretBoard.php를 생성하였다. 해당 부분에서는 패스워드를 입력받는데 no에 대한 정보의 유치를 위해서 hidden 타입으로 추가하였다. 이제 POST데이터로 secretPassword를 받은 내용을 처리해주는 코드를 view에 추가하면 된다.

```php
    if(isset($_REQUEST['no'])){
      $no=fix_string($_REQUEST['no']);
    }

    if(isset($_REQUEST['secretPassword'])){
      $secretKey=fix_string($_REQUEST['secretPassword']);
    }
```

기존의 GET파라미터 형식의 no를 REQUEST로 바꿈으로써 get과 post형식을 동시에 지원하게 된다. 

```php
    if($row['secret'] == 1){
      if(strlen($secretKey)<1){
        echo '<script>alert("비밀글입니다.");';
        echo "location.href='/secretBoard.php?no={$no}';</script>";
        exit;
      }

      if($row['secretPassword'] == $secretKey){
        echo '<script>alert("right!"");</script>';
      }

      else{
        echo '<script>alert("패스워드를 잘못 입력하셨습니다.");';
        echo "location.href='/secretBoard.php?no={$no}';</script>";
        exit; 
      }
    }
```

그리고 비밀글 부분의 루틴에서 password를 입력받았는지를 판단한 뒤 쿼리 조회결과와 비교하여 같은 경우에는 밑의 루틴을 실행하며 다른 경우에는 다시 secretBoard.php로 리다이렉트를 하도록 하였다. 이제 해당 부분에서 password와 같이 평문형식으로 오가는 값에 대한 조건 변경을 아래와 같이 진행하면 대략적인 구현이 마무리된다.

```php
    if($row['secret'] == 1){
      if(strlen($secretKey)<1){
        echo '<script>alert("비밀글입니다.");';
        echo "location.href='/secretBoard.php?no={$no}';</script>";
        exit;
      }

      if(!password_verify($row['secretPassword'], $secretKey)){
        echo '<script>alert("패스워드를 잘못 입력하셨습니다.");';
        echo "location.href='/secretBoard.php?no={$no}';</script>";
        exit;
      }
    }
```

### 게시판 글 수정 구현

글에 대해 수정하는 루틴을 작성하고자 하는데 가장 좋은건 view 페이지 내부에서 리다이렉션을 하는 방법이다. 그러기 위해서는 작성자가 맞는지 검증이 필요하다. 

```php
<button type="button" onclick="location.href='javascript:history.back();'">뒤로가기</button>
<button type="button" onclick="location.href='/modifyPage.php?no=<?php echo $no?>'">수정하기</button>
```

view.php에서 modifyPage.php로의 링크를 추가해준다.

```php
<?php
  session_start();
  $title = "글 수";
  require_once('head.php');
  if(isset($_SESSION['USERSESSION'])){
  echo '로그인 정보 ' . $_SESSION['NICKNAME'] . '<br>';
  echo '<a href="/logout.php">로그아웃</a>';
  }
?>

<?php
  if(!isset($_SESSION['USERSESSION'])){
    echo '<script>alert("로그인이 필요합니다.");';
    echo 'location.href="/loginForm.php";</script>';
    exit;
  }

  function fix_string($string){
    if(get_magic_quotes_gpc()) $string=stripslashes($string);
    return htmlentities($string);
  }
  
  $no="";
  if(isset($_REQUEST['no'])){
    $no=fix_string($_REQUEST['no']);
  }

  if($no<=0 or !$no){
    echo '<script>alert("잘못된 접근입니다!");';
    echo 'history.back();</script>';
    exit;
  }

  require_once('BoardDBConnect.php');
  $query = "SELECT * FROM FreeBoard WHERE boardNo LIKE ?";
  $stmt = $conn->stmt_init();
  $stmt = $conn->prepare($query);
  $stmt->bind_param("i", $no);
  $stmt->execute();
  $result = $stmt->get_result();
  $row = mysqli_fetch_array($result);

  if(!$row){
    echo '<script>alert("잘못된 접근입니다!");';
    echo 'history.back();</script>';
    exit;
  }

  if(!($row['author'] == $_SESSION['nickname'])){
    echo '<script>alert("글 작성자만 수정이 가능합니다.");';
    echo 'history.back();</script>';
    exit;
  }

?>

<form action="modify.php" method="POST">
<input type="hidden" id="no" name="no" value="<?php echo $no?>">
<input type="text" name="title" value="<?php echo $row['title'] ?>" style="width:300px;"><br>
<input type="text" name="content" value="<?php echo $row['content'] ?>" style="width:300px;height:200px;"><br>
<input type="password" name="password" placeholder="패스워드" required><br>
<button type="submit">수정</button>
<button type="button" onclick="location.href='javascript:history.back();'">뒤로가기</button>
</form>
```

로직은 no를 넘겨준 뒤 쿼리를 조회한다. 그리고 정보가 존재하는 경우 author와 userID가 동일한지를 판단한 뒤 동일한 경우 내용을 form의 형태로 보여주게 된다. 그리고 password를 포함한 수정된 정보를 modify.php로 전송하면 해당 부분에서 검증 후에 업데이트가 진행되도록 할 것이다.

```console
mysql> select * from FreeBoard;
+---------+-------------+--------+---------------------+---------+--------+----------------+------------+
| boardNo | title       | author | time                | content | secret | secretPassword | permission |
+---------+-------------+--------+---------------------+---------+--------+----------------+------------+
|       1 | HelloWorld  | Admin  | 2020-02-20 00:00:00 | ABCDEF  |      0 | NULL           |          1 |
|       2 | HelloWorld2 | Admin  | 2020-02-20 00:00:00 | ABCDEF  |      0 | NULL           |          1 |
|       3 | HelloWorld2 | Admin  | 2020-02-20 00:00:00 | ABCDEF  |      0 | NULL           |          2 |
|       4 | HelloWorld2 | Admin  | 2020-02-20 00:00:00 | ABCDEF  |      0 | NULL           |          3 |
|       5 | HelloWorld2 | Admin  | 2020-02-20 00:00:00 | ABCDEF  |      1 | abcdqwer       |          2 |
+---------+-------------+--------+---------------------+---------+--------+----------------+------------+
```

다음과 같이 게시판이 있는데 UPDATE를 사용하여 수정할 것이다.

```console
mysql> UPDATE FreeBoard SET permission='2', author='abcd' WHERE boardNo='5';
Query OK, 1 row affected (0.00 sec)
mysql> select * from FreeBoard;
+---------+-------------+--------+---------------------+---------+--------+----------------+------------+
| boardNo | title       | author | time                | content | secret | secretPassword | permission |
+---------+-------------+--------+---------------------+---------+--------+----------------+------------+
|       1 | HelloWorld  | Admin  | 2020-02-20 00:00:00 | ABCDEF  |      0 | NULL           |          1 |
|       2 | HelloWorld2 | Admin  | 2020-02-20 00:00:00 | ABCDEF  |      0 | NULL           |          1 |
|       3 | HelloWorld2 | Admin  | 2020-02-20 00:00:00 | ABCDEF  |      0 | NULL           |          2 |
|       4 | HelloWorld2 | Admin  | 2020-02-20 00:00:00 | ABCDEF  |      0 | NULL           |          3 |
|       5 | HelloWorld2 | abcd   | 2020-02-20 00:00:00 | ABCDEF  |      1 | abcdqwer       |          2 |
+---------+-------------+--------+---------------------+---------+--------+----------------+------------+
5 rows in set (0.00 sec)
```

UPDATE 명령어로 Table을 선택한 뒤 SET을 통해 변경하고자 하는 필드를, WHERE를 통해 조건을 주면 된다.

```php
<?php
  session_start();
  $title = "글 수";
  require_once('head.php');

  function fix_string($string){
    if(get_magic_quotes_gpc()) $string=stripslashes($string);
    return htmlentities($string);
  }

  $id=$_SESSION['USERSESSION'];
  $no=$pw=$title=$content="";
  if(isset($_REQUEST['password']))
    $pw=fix_string($_REQUEST['password']);

  if(isset($_REQUEST['title']))
    $title=fix_string($_REQUEST['title']);

  if(isset($_REQUEST['content']))
    $content=fix_string($_REQUEST['content']);

  if(isset($_REQUEST['no'])){
    $no=fix_string($_REQUEST['no']);
  }

  if($no<=0 or !$no){
    echo '<script>alert("잘못된 접근입니다!");';
    echo 'history.back();</script>';
    exit;
  }

  if(strlen($pw)<4){
    echo '<script>alert("패스워드를 다시 입력해주세요.");';
    echo 'history.back();</script>';
    exit;
  }

  require_once('UserDBconnect.php');
  $query = "SELECT * FROM Account WHERE userID LIKE ?";
  $stmt = $conn->stmt_init();
  $stmt = $conn->prepare($query);
  $stmt->bind_param("s", $id);
  $stmt->execute();
  $result = $stmt->get_result();
  $row = mysqli_fetch_array($result);

  if(!$row){
    echo '<script>alert("잘못된 정보입니다.");';
    echo 'history.back();</script>';
    exit;
  }

  if(!($row['userID']==$id)){
    echo '<script>alert("잘못된 정보입니다.");';
    echo 'history.back();</script>';
    exit;
  }

  if(!password_verify($pw, $row['password'])){
    echo '<script>alert("패스워드가 잘못되었습니다.");';
    echo 'history.back();</script>';
    exit;
  }

  $conn2 = new mysqli("db", "wizley", "alpine", "Board");
  if(!$conn2){
    die("Connection Error!");
  }
  $query = "SELECT * FROM FreeBoard WHERE boardNo LIKE ?";
  $stmt = $conn2->stmt_init();
  $stmt = $conn2->prepare($query);
  $stmt->bind_param("i", $no);
  $stmt->execute();
  $result = $stmt->get_result();
  $row = mysqli_fetch_array($result);

  if(!$row){
  echo '<script>alert("잘못된 접근입니다!");';
  echo 'history.back();</script>';
  exit;
  }

  if(!($row['author'] == $_SESSION['nickname'])){
    echo '<script>history.back();</script>';
    exit;
  }

  $query = "UPDATE FreeBoard SET title=?, content=? WHERE boardNo=?";
  $stmt = $conn2->stmt_init();
  $stmt = $conn2->prepare($query);
  $stmt->bind_param("ssi", $title, $content, $no);
  $stmt->execute();

  $stmt->close();
  $conn->close();
  $conn2->close();

  echo '<script>alert("수정이 완료되었습니다.");';
  echo 'history.go(-2);</script>';
?>
```

들어온 정보를 수정하는 modify에서는 각각의 POST 필드에 대한 값 검사를 진행한 뒤, 넘어온 패스워드를 쿼리 조회를 통해 비교한다. 그 후 쿼리 조회를 통해 변경하고자 하는 게시물의 값을 가져와 author의 정보가 SESSION의 닉네임 정보와 일치하는지 한번 더 검사를 진행한 뒤에 UPDATE를 통해 수정을 하게 된다. 그 후 2페이지 전으로 go를 통해 돌아가는 루틴을 가지게 된다. 

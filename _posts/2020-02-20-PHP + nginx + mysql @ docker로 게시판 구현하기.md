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
    index index.html;
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

## User DB 구현 및 페이지 생성

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
		echo 'location.href="/loginForm.html";</script>';
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
		echo 'location.href="/loginForm.html";</script>';
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
		echo 'location.href="/loginForm.html";</script>';
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
		echo 'location.href="/loginForm.html";</script>';
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











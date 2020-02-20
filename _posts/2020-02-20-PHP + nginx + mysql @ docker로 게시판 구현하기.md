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

![helloworld](https://raw.githubusercontent.com/wizleysw/wizleysw.github.io/master/_posts/img/php_board/helloworld.PNG)

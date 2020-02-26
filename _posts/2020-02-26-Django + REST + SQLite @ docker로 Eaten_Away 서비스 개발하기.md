---
published: true
layout: single
title : "Django + REST + SQLite @ docker로 Eaten_Away 서비스 개발하기"
category : dev
comments: false
author_profile : true
tag : 
  - Django
  - REST
  - docker
toc : true
---

## Overview

방금 PHP 개발 프로젝트를 마무리하고 바로 시작하게 된 다음 프로젝트는 요즘 젤 잘 나가는 강력한 프레임워크인 Django를 사용한다. 그리고 이전 포스팅은 간단한 게시판을 구현한 정도였다면 이번에는 하나의 서비스를 개발한다는 생각으로 진행할 예정이다. 이번에도 docker를 사용할 것인데 도커 이외에 Django + REST + SQLite라는 또다시 나에게 생소한 조합으로 개발을 진행하고자 한다.

이번에 서비스의 주제로 정한 키워드는 "음식"이다. 내 일과를 생각해보았을 때, 아침, 점심, 저녁에 뭘 먹을지에 대한 고민이 꽤 큰 비율을 차지한다. 그래서 이번 프로젝트는 다음의 기능을 가지고 있는 서비스를 개발하고자 한다.

1. RANDOM으로 추천메뉴를 가져온 뒤 상점의 리스트를 사용자에게 보여준 뒤 하나를 선택할 시 상세정보를 확인(맵, 평점 등)
2. 먹고 싶은 메뉴 등의 조건필터링 후의 결과 확인
3. 개개인이 먹은 메뉴 정보를 저장하여 한달동안 어떤 음식을 몇번이나 또는 얼마만의 주기마다 먹었는지를 확인하여 그래프 등으로 표시하는 기능

그리고 해당 개념을 토대로 구현하고자 하는 세부적인 목표를 정하자면 다음과 같다.

1. 회원가입 / 로그인 -> 메일 인증 기능
2. 개인정보 열람(본인에 의하여/타인에 의하여) 및 수정
3. 게시물의 댓글 + 대댓글 기능 구현
4. 맵 관련 api 연동
5. 실시간 대화 or 1:1 대화방 구현

약간 배달 어플과 흡사한 기능을 가지고 있다고 볼 수도 있을 것 같다. 어쩌면 거의 같을지도 모르겠다. 적는 지금 생각을 해보니 음식점에 대한 정보를 업데이트 하기 위해서 DB에서 사용자에 대한 정보를 다음과 같이 세분화 할 수 있을 것 같다.

1. 사용자
2. 점주
3. 관리자

세부적인 사항은 DB를 설계하게 되면 그 때 더 심층적으로 고민을 해보도록 하고 지금해야할 것은 REST api와 Docker에 대해 이해하는 것과 그에 따라 docker-compose파일을 작성하는 것이다. 그러면 먼 여정을 시작해보도록 하자.

## Settings

### Django

장고는 MVT 구조를 채택하고 있다. 아래의 링크를 통해 해당 부분을 공부할 수 있다.

[Django MVT 패턴](https://butter-shower.tistory.com/49?category=718374)

장고는 python 기반으로 구현되었기 때문에 python 문법을 알면 개발이 가능하다는 특장점이 있으며, 정해진 규칙에 의거하여 개발을 진행해야 하기 때문에 딴 사람의 코드에 대한 이해가 쉽고 개발을 오래하여 구현에 익숙해지면 쉽다는 장점이 있다. 

### REST FrameWork

이 주제도 아래의 링크에 잘 설명되어 있다.

[REST API란](https://gmlwjd9405.github.io/2018/09/21/rest-and-restful.html)

얘를 왜 쓰고싶었냐면 안드로이드/웹 등의 멀티 플랫폼 개발에서 효율적으로 사용될 뿐만 아니라 POST/GET 등의 HTTP 메소드를 통해 API를 구현하기 때문에 형식적이고 간단하다는 느낌이 들었다. 아직 제대로 활용을 안해봤기 때문에 사용하면서 느껴봐야 할 것 같다.

### Dockerfile 

이번에는 Docker에 Django 세팅이 필요하다. python2의 지원이 만료되는 시점에서 python3를 사용하는게 좋을것같기에 Ubuntu 이미지 내부에 Django 및 REST framework를 설치하는 방식을 사용하기로 하였다. 자 그럼 도커파일을 먼저 만들어야 되는데 Django같은 경우 Ubuntu 또는 Python 이미지를 베이스로 생성을 한다. python3 이미지를 사용하면 pip / python3에 대한 버전관리가 되며 이미지 용량이 Ubuntu보다 가볍기 때문에 Python 이미지를 베이스로 사용하는 것이 좋을 것 같았다.

그래서 다음과 같이 초기 Dockerfile을 만들었다.

```console
FROM python:3

MAINTAINER Wizley <wizley@kakao.com>

WORKDIR /code

RUN pip3 install \
	django \
	django-cors-headers \
	djangorestframework \
	djangorestframework-jwt

ENTRYPOINT \
	django-admin startproject eatenAway  \ 
	python3 $(pwd)/eatenAway/manage.py makemigrations && \
	python3 $(pwd)/eatenAway/manage.py migrate && \
	echo "from django.contrib.auth import get_user_model; User = get_user_model(); User.objects.create_superuser('root', 'wizley@github.com', 'alpine')" | python3 $(pwd)/eatenAway/manage.py shell && \
	python3 $(pwd)/eatenAway/manage.py runserver 0.0.0.0:8000
```

이제 아래의 명령어로 image를 생성한다.

```console
Wizley:~/Project/Django/eatenAway # docker build --tag django:1.0 .
Successfully built 18300e8ec004
Successfully tagged django:1.0

Wizley:~/Project/Django/eatenAway # docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
django              1.0                 18300e8ec004        2 minutes ago       976MB
```

확인을 하기 위해서 container로 해당 이미지를 올려볼 것이다. 일단은 테스트이기 때문에 abc라는 이름으로 컨테이너를 올린 뒤 접속해보았다.

```console
Wizley:~/Project/Django/eatenAway # docker run -it -d --name abc -p 8000:8000 django
fd826b9f72a23f08d2c7755e2bca48d46397d6ea28e9f154ff9e233d7ea15c2c

Wizley:~/Project/Django/eatenAway # docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
fd826b9f72a2        django                "/bin/sh -c 'django-…"   7 seconds ago       Exited (2) 6 seconds ago                       abc
```

Exited 상태가 떴다. 예감이 안좋다. 명령어상에 문제가 없어서 docker logs 명령어로 문제의 발생지점을 찾아보았다.

```console
Wizley:~/Project/Django/eatenAway # docker logs abc
usage: django-admin startproject [-h] [--template TEMPLATE]
                                 [--extension EXTENSIONS] [--name FILES]
                                 [--version] [-v {0,1,2,3}]
                                 [--settings SETTINGS]
                                 [--pythonpath PYTHONPATH] [--traceback]
                                 [--no-color] [--force-color]
                                 name [directory]
django-admin startproject: error: unrecognized arguments: /code/eatenAway/manage.py makemigrations
```

arguments 결과가 잘못들어간 것을 확인할 수 있었다!! 적는 과정에서 startproject뒷 부분에 &&을 까먹고 넣지 않았던 것이다. 해당 부분을 수정하고 다시 빌드를 한 뒤 같은 명령어로 실행시켜보았다.


```console
Wizley:~/Project/Django/eatenAway # docker run -it -d --name abc -p 8000:8000 django:1.0
310f35fcfdb2743a4616f7d0a38f8d7a3a73c06b99e9162db9129cd5872e8873

Wizley:~/Project/Django/eatenAway # docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS                    NAMES
310f35fcfdb2        django:1.0          "/bin/sh -c 'django-…"   3 seconds ago       Up 2 seconds               0.0.0.0:8000->8000/tcp   abc
```

제대로 올라간 것을 확인가능하다. 이제 shell을 붙어서 상태를 확인해보겠다.

```console
root@310f35fcfdb2:/code/eatenAway# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 11:07 pts/0    00:00:00 /bin/sh -c django-admin startproject eatenAway && ?python3 $(pwd)/eate
root        20     1  0 11:07 pts/0    00:00:00 python3 /code/eatenAway/manage.py runserver 0.0.0.0:8000
root        22    20  5 11:07 pts/0    00:00:03 /usr/local/bin/python3 /code/eatenAway/manage.py runserver 0.0.0.0:808
root        25     0  0 11:08 pts/1    00:00:00 /bin/bash
root        36    25  0 11:09 pts/1    00:00:00 ps -ef
```

이제 localhost:8000을 접속해보도록 하자.

![startproject](https://raw.githubusercontent.com/wizleysw/wizleysw.github.io/master/_posts/img/eatenAway/startproject.png)


빙고! 장고 프로젝트가 정상적으로 시작된 것을 확인할 수 있다. 여기서 끝내지 않고 조금 더 나아가서 requirements.txt로 패키지의 내용들을 분리한 뒤 실행하여 설치하는 방식으로 수정하도록 하겠다.

### requirements.txt

pip의 freeze라는 명령어를 사용하면 설치된 패키지를 알파벳 순으로 출력하는 것이 가능하다. 이를 통해 버전에 대한 정보를 가져올 수 있다.

```console
root@de8d3ccac6db:/code# pip3 freeze > requirements.txt
root@de8d3ccac6db:/code# ls
eatenAway  requirements.txt
root@de8d3ccac6db:/code# cat requirements.txt
asgiref==3.2.3
attrs==19.3.0
coverage==5.0.3
Django==3.0.3
django-cors-headers==3.2.1
djangorestframework==3.11.0
djangorestframework-jwt==1.11.0
more-itertools==8.2.0
packaging==20.1
pluggy==0.13.1
py==1.8.1
PyJWT==1.7.1
pyparsing==2.4.6
pytest==5.3.5
pytest-cov==2.8.1
pytz==2019.3
six==1.14.0
sqlparse==0.3.0
wcwidth==0.1.8
```

이제 도커를 다음과 같이 변경을 한다. 이제 requirements.txt를 바탕으로 패키지에 대한 설치가 진행되도록 한 것이다.

```console
FROM python:3

MAINTAINER Wizley <wizley@kakao.com>

WORKDIR /code
COPY ./requirements.txt /code

RUN pip3 install -r ./requirements.txt

ENTRYPOINT \
	django-admin startproject eatenAway && \
	python3 $(pwd)/eatenAway/manage.py makemigrations && \
	python3 $(pwd)/eatenAway/manage.py migrate && \
	echo "from django.contrib.auth import get_user_model; User = get_user_model(); User.objects.create_superuser('root', 'wizley@github.com', 'alpine')" | python3 $(pwd)/eatenAway/manage.py shell && \
	python3 $(pwd)/eatenAway/manage.py runserver 0.0.0.0:8000

```

이제 다시 image를 빌드한 뒤 run을 시키게 되면 기본 container를 돌릴 수 있다.

### docker-compose

자 이제는 compose 파일을 만들차례이다. 이번에는 PHP 프로젝트와 달리 일단은 하나의 컨테이너만을 사용할 예정이기 때문에 network 설정은 하지 않도록 한다. 
(여기서 엄청나게 삽질을 시작하였다...)

지금까지 ENTRYPOINT로 호출했었는데 한번만 startproject가 실행되어야 하므로 결론적으로 Dockerfile을 다음과 같이 변경하였다.

```console
FROM python:3

MAINTAINER Wizley <wizley@kakao.com>

WORKDIR /code
COPY ./requirements.txt /code
COPY ./docker-entrypoint.sh /code

RUN pip3 install -r ./requirements.txt
```

그리고 docker-compose를 다음과 같이 작성하였다.

```console
version: '3'

services:
 django:
  container_name: django_eatenAway
  image: django:1.0
  environment:
    - DJANGO_DEBUG=true
  volumes:
    - ./code:/code
  ports:
    - "8000:8000"
  tty: true
  command: python3 ./eatenAway/manage.py runserver 0:8000
 ```

 맨처음에는 command가 없이 실행을 한 뒤 code 내부에 옮겨둔 docker-entrypoint.sh를 실행시킨다.

 ```sh
django-admin startproject eatenAway
python $(pwd)/eatenAway/manage.py makemigrations
python $(pwd)/eatenAway/manage.py migrate
echo "from django.contrib.auth import get_user_model; User = get_user_model(); User.objects.create_superuser('root', 'wizley@github.com', 'alpine')" | python3 $(pwd)/eatenAway/manage.py shell
```

그 후에 command를 추가하는 방식으로 진행하면 빌드에 성공할 수 있다. 자 이제 다시 명령어를 입력한다.

```console
docker build --tag django:1.0 . ; docker-compose up

Recreating django_eatenAway ... done
Attaching to django_eatenAway
django_eatenAway | Python 3.8.1 (default, Feb  2 2020, 08:37:37)
django_eatenAway | [GCC 8.3.0] on linux
django_eatenAway | Type "help", "copyright", "credits" or "license" for more information.
```

그리고 컨테이너 내부로 접속한다.

```console
Wizley:~/Project/Django/eatenAway # cat shell.sh
docker exec -it django_eatenAway /bin/bash
Wizley:~/Project/Django/eatenAway # sh shell.sh
root@f7aadb20f3c1:/code#
```

code에 넣어둔 docker-entrypoint.sh 파일을 실행시키게 되면 

```console
root@f7aadb20f3c1:/code# sh docker-entrypoint.sh
No changes detected
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying sessions.0001_initial... OK

root@f7aadb20f3c1:/code# ls -al
total 8
drwxr-xr-x 4 root root  128 Feb 26 14:36 .
drwxr-xr-x 1 root root 4096 Feb 26 14:35 ..
-rw-r--r-- 1 root root  319 Feb 26 14:15 docker-entrypoint.sh
drwxr-xr-x 5 root root  160 Feb 26 14:36 eatenAway
```

project로 eatenAway가 생성되는 것을 확인할 수 있다. 이제 컴포즈에 윗 부분의 runserver를 추가하면 아래와 같이 up을 통하여 서버를 구동할 수 있다.

```console
Wizley:~/Project/Django/eatenAway # docker-compose up
Recreating django_eatenAway ... done
Attaching to django_eatenAway
django_eatenAway | Watching for file changes with StatReloader
django_eatenAway | Performing system checks...
django_eatenAway |
django_eatenAway | System check identified no issues (0 silenced).
django_eatenAway | February 26, 2020 - 14:37:39
django_eatenAway | Django version 3.0.3, using settings 'eatenAway.settings'
django_eatenAway | Starting development server at http://0:8000/
django_eatenAway | Quit the server with CONTROL-C.
```

이제 기본적인 docker 세팅이 끝났다.
---
published: true
layout: single
title : "PHP + nginx + mysql @ docker로 게시판 구현하기"
category : dev
comments: false
tag : PHP
---

## Overview

이번 프로젝트의 목표는 크게 3가지로 나눌 수 있을 것 같다.

첫 번째 목표는 웹개발을 해본다는 것이다. PHP는 대학교를 첫 입학했던 시기에는 가장 많이 사용하는 웹 개발 언어였다. 하지만 지금은 React, Django 등의 더 사용하기 쉽고 강력한 프레임웍들이 나왔고 php는 개발자들이 유지보수에 적극적이지 않다는 이유로 점차 사용되지 않는 언어가 되어가고 있다. 하지만 기존에 구현된 수많은 사이트는 php를 사용하기 때문에 한 번쯤은 사용을 해보는 것이 좋다고 생각하였다. 

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
위에서 언급했듯이 올해이 첫 프로젝트부터 Docker를 활용하기로 맘을 먹었다. 그리고 나는 Mac을 사용하기 때문에 Mac에서 도커를 생성하여 개발을 진행할 것이다. 게시판은 php, nginx, mysql의 조합으로 개발을 진행할 것인데 mysql은 DB로 사용되는 것을 알지만 nginx가 왜 필요한지에 대한 의문이 들었다. 

### nginx

nginx는 동시접속을 효율적으로 처리하기 위해 개발이 된 웹 서버 프로그램이며 동시접속자의 수가 700명을 초과할 경우 서버를 증설하거나 nginx를 도입한다고 한다. nginx는 크게 두 가지 역할을 가지고 있는데 첫 번째 역할이 HTTP 서버이다. Html, CSS, Javascript등의 정적인 정보들을 앞단에서 처리해줌으로써 서버의 과부하를 줄이는데 사용됨을 의미한다. 두 번째로는 리버스 프록시 서버로서의 역할인데 클라이언트가 웹 서버로 바로 요청을 하는 것이 아닌 nginx서버를 요청을 하고 nginx 서버에서 요청을 전달하도록 하는데, 이 상황에서 nginx가 여러 서버로 분산해서 요청을 전달하는 로드 밸런싱을 위해 사용되기도 한다는 것이다. 이를 통해 서버에 대한 노출이 최소화되며 layer가 한단계 더 추가된다는 장점이 있다. 또 하나의 특징으로는 비동기 처리 방식을 채택한다는 점을 들 수 있는데, 이로 인하여 1개의 연결당 하나의 쓰레드를 할당하는 방식이 아닌 Event handler가 다수의 연결을 비동기로 처리해 한정된 자원으로 처리가 가능하다. 해당 부분에 대한 설명은 아래의 링크를 참조하면 좋을 것 같다.

[readdir과 readdirSync 성능비교와 비동기 그리고 동기](https://blog.naver.com/jhc9639/221108496101)

그러면 이제 Docker를 설치해야 되는데 mysql, php, nginx를 각각 다른 Container로 받아오는 작업을 할 차례이다. latest 태그를 통해 최신 버전을 받을 수 있지만 개발 환경을 통일하기 위해 모든 컨테이너를 특정 버전을 명시하여 받을 것이다.

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

마찬가지로 같은 방식으로 php와 mysql의 Container를 다운받는다.

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
php                 7.4                 7dc31b4f3403        2 weeks ago         405MB
mysql               8.0.19              791b6e40940c        2 weeks ago         465MB
```

이제 제대로 받아왔는지 확인을 하기 위해 php 컨테이너에 직접 접속을 해보도록 하겠다.

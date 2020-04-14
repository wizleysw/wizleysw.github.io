---
published: true
layout: single
title : "JAVA + graphQL로 안드로이드 어플 클론코딩하기"
category : dev
comments: false
author_profile : true
tag : 
  - Android
  - graphQL
  - JAVA
toc : true
---

## Overview

장고로 프로젝트가 마무리되자 말자 선택한 프로젝트는 안드로이드 클론 코딩이다. 개발자들이 많이 쓰는 JAVA라는 언어에 대해서 전혀 몰랐기 때문에 나중에 Spring으로 개발을 진행하는 상황을 대비할겸, 가장 핫한 주제인 모바일에 관련된 어플을 만들어보고싶기도했다. 사실 안드로이드를 개발해본적은 없지만 해킹 및 안드로이드 어플을 리버싱하는 과정을 통해서 대략적으로 어떻게 생겼는지에 대한 정보는 알고 있지만 정확히 어떤 식으로 안드로이드가 동작하는지에 대한 개념은 머리에 정립되어 있지 않았다. 이번 기회에 해킹이 아닌 개발의 관점에서 안드로이드 어플을 개발해보고 싶었다. 최근 코틀린으로 앱이 많이 개발되고 있기는 하지만 자바로도 짜보지 않은 앱을 코틀린으로 짜보기보다는 자바로 먼저 짠 뒤에 코틀린으로 포팅하는 방식이 더 효율적이라고 생각이 들었다. 그래서 요 근래 며칠간 자바의 정석이라는 책을 속독하였는데 내용이 너무 좋았다. 아무래도 low단에서 분석을 많이 진행하였다보니 책에서 저자가 설명한 메모리 그림들이 쏙쏙 이해가 잘 되었다. 

이번에는 어떤 것들을 공부해볼까라는 생각을 하다보니 크게 4가지 정도가 머리속에서 떠올랐다. 매번 새로운 것들을 시도한다는 것은 여간 고통스러운것이 아니지만 하고나서 결과물을 볼 때의 그 성취감이 매번 도전을 하도록 만든다. 

1. graphQL로 API 구현하기
2. JAVA로 안드로이드 개발
3. redis 사용해보기
4. 네이버/카카오 계정 로그인 구현

이 전 프로젝트의 주제였던 eaten-Away 서비스를 만드는 과정에서 REST로 api 서버를 설계하였었다. GET, POST 등의 HTTP 메소드와 연동하는 부분이 편리하고 좋았지만 비슷한 기능을 가진 여러 기능들이 각각 매칭이 되어야한다는 불편함이 있었다. 이런 몇가지 불편함을 해소하기 위해서 facebook에서 graphQL을 고안하였다고 하는데 직접 사용을 해보면서 어떤 점이 개선이 되었는지를 비교해보고 싶었다. JAVA의 경우 앞서 말했듯이 Spring으로 개발을 할 프로젝트가 준비되어 있기 때문에 이를 위해서 미리 JAVA를 숙달한다는 의미가 있는 것 같다. (여담이지만 알고리즘에 대한 다른 사람의 풀이를 보면 C++ 뿐만 아니라 JAVA로 작성된 코드들이 많은데 책 한권 봤다고 이해가 되더라.) redis 같은 경우는 캐시 서버?? 관련 개념으로 자주 등장하는 것 같은데 아무래도 개발초보다 보니 멀티 쓰레드/DB/캐시 관련 지식이 부족하다보니 왜 저런 애들이 추가로 쓰이는지가 궁금하였다. 그래서 만약 안드로이드 개발에 사용될 수 있다면 한 번 같이 사용해보는 것도 괜찮다고 생각이 들었다. 마지막으로 네이버/카카오 계정 로그인 구현의 경우 요즘 대부분의 앱들이 따로 회원가입을 처리하지 않고 해당 api를 사용하기도 하고, 회원가입 및 개인정보와 관련된 부분을 덜 신경쓸 수 있다는 점에서 사용을 해보기로 마음먹었다. 어차피 클론코딩이고 어플에서도 api로 구현하긴 하였다.

### AppName

클론 코딩의 주제로 정한 어플은 "세줄일기"라는 어플이다.

[인스타그램](https://play.google.com/store/apps/details?id=com.instagram.android&hl=ko)

원래 하고자 하는 다른 어플이 있었는데, 메일로 문의한 내용에 대한 답변이 오지 않아 부득이하게 변경을 하게 되었다. 많은 사람들이 클론 코딩 주제로 인스타그램을 선정하였는데 nodejs, react가 아닌 JAVA를 토대로 비슷하게 흉내내보는 토이 프로젝트로 진행하기로 하였다.

## JAVA 핵심 정리

자바를 공부하는 처음 접하거나 헷갈리는 개념들이 몇 있었다. 그 개념들에 대해서 간략하게 이해한대로 정리를 해놓고자 한다.

### 제어자(Modifier)

제어자는 클래스나 변수 등의 선언부에 함께 사용하여 의미를 나타내는데 하나의 대상에 여러 조합을 사용하는 것이 가능하다. 하지만 public, protected, default, private을 의미하는 접근 제어자는 한 가지 종류만 사용이 가능하다.

### static

static은 클래스 내에 생성이 될 경우 인스턴스에 관계없이 같은 값을 가지게 된다. 이는 static 클래스 변수를 모든 인스턴스가 공유함을 의미한다. 그 외에 static이 붙은 변수나 메서드 등은 인스턴스를 생성하지 않고 사용이 가능하다. 이 경우에 만약 메소드를 static으로 생성하였다면 인스턴스 멤버를 직접 사용하는 것이 불가능하다.

```java
class Student{
	static int grade = 90;

	static {
		// 스태틱 변수의 초기화 수행
	}

	static int is_bigger(int a, int b){
		return a>b;
	}
}
```

### final

final은 용어에서 예상이 가듯, 변경할 수 없다는 의미를 주기 위해 사용한다. 변수에 final을 붙히게 되면 상수가 되며 메서드에 붙히면 오버라이딩을 지원하지 않음을 의미한다. final은 클래스, 메서드, 멤버변수, 지역변수 모두에 사용이 가능하다.

```java
final class Wizley{
	final int grade = 100;

	final void getGrade(int grade){
		return grade;
	}
}
```

위와 같이 Wizley라는 클래스를 final로 선언을 하면 다른 클래스의 조상 클래스가 될 수 없으며 내부 값인 grade의 값 변경이 불가능하며 getGrade의 경우 오버라이딩이 불가능하게 된다.

만약 인스턴스의 변수가 final일 경우 생성자에서 초기화를 수행한다. 

```java
class Student{
	final int grade;

	Student(int gradeinfo){
		grade = gradeinfo;
	}

	Student(){
		this(0);
	}
}

class Test{
	public static void main(String args[]){
		Student A = new Student(100);
	}
}
```

이렇게 클래스 내부에서 값을 설정하지 않은 경우 생성자를 통해 초기화가 가능하며 그 후에는 값에 대한 변경이 불가능하다.

### abstract

abstract는 추상적인 형태를 표현하기 위해 클래스 또는 메소드에 사용된다. 선언부만 작성을 한 뒤에 구현을 나중에 작성할 시에 사용된다.

```java
abstract class AbstractStudent {
	abstract void getGrade();
}
```

### public

public은 접근 제어자 중 하나이며 멤버 또는 클래스에 사용된다. 이런 접근 제어자들은 멤버 또는 클래스에 대한 정보를 외부에서 접근하지 못하도록 범위를 제한하는 경우에 주로 사용하며 default가 기본적으로 설정되어 있다. 이런 접근 제어자들 중 public은 제한이 전혀 없는 경우를 의미한다. 이는 즉 어디에서는 접근이 가능하다는 것을 의미한다. 

### default

기본 옵션으로 설정되는 default는 같은 패키지 내에서만 접근이 가능하다. 

### protected

default보다 조금 더 넓은 범위로 같은 패지기 뿐만 아니라 다른 패키지의 자손클래스에서도 접근이 가능하다.

### private

같은 클래스 내부에서만 접근이 가능하다. 또 private이 생성자에 추가된 경우 인스턴스의 생성을 제한할 수 있으며 생성을 위해서는 내부에서 생성에 사용할 static 메소드를 선언하여 호출하면 된다.

```java
class Student{
	private static Student A = new Student();

	private Student(){

	}

	public static Student getStudent(){
		return A;
	}

}
```

지금까지의 접근 제어자의 접근 권한을 범위로 표현하면 다음과 같다.

```
public > protected > default > private
```

이런 특징에 의해 클래스는 보통 public 또는 default 중 하나로 생성이 되며 메서드 또는 멤버변수의 경우 4가지 옵션이 다 사용된다. 궁극적으로 이런 범위 제한은 클래스 내의 특정 정보를 외부에서 접근하지 못하도록 하기 위함이며 이는 캡슐화를 위함이다. 즉, 외부로부터 데이터를 보호하기 위해서 사용된다.

### 인터페이스(Interface)

추상 클래스와 거의 비슷한데 추상 메소드 또는 상수가 아닌 값을 가질 수 없다는 특징이 있다. 

```java
interface Movable{
	public static final int A = 0;
	public abstract move();
}
```

이런 특징으로 인해 멤버변수의 경우 무조건 public static final 이어야 하며 메소드는 public abstract이여야 한다. 만약 생략할 경우 자동으로 설정이 된다. 

자바는 클래스에 대한 다중상속이 불가능하지만 인터페이스는 가능하다.

```java
interface Foodeatable extend Drinkable, Eatable, Movable {}

class Student implements Foodeatable{
	public void move(){

	};
	...
}
```

인터페이스로부터 상속받은 메소드들에 대하여 클래스 내부에서 그에 맞게 구현을 해주어야 된다. 이런 점을 활용하여 다중상속을 구현하는 것이 가능하고 구현 부분이 인터페이스 내부에 존재하지 않기 때문에 그로 인하여 문제가 발생할 확률이 적다. 자바의 특성상 클래스를 구현하는 과정에서 다중 상속을 위해서는 다른 클래스를 클래스 멤버로 생성하여야 되는데 인터페이스로 이런 부분에 대한 효율적인 해결이 가능하다. 또한 서로 관계가 없는 클래스가 여러개 존재할 경우 인터페이스를 통해 상속관계가 아님에도 공통적인 구현이 가능하다는 특징이 있다.

이 점은 자바의 정석에 좋은 예제가 있는데 Unit이라는 클래스의 하위 클래스로 GroundUnit과 AirUnit이 있다고 가정해보자. 만약 SCV라는 Unit이 GroundUnit의 Tank와 AirUnit의 Dropship에 대해서 Repair가 가능한 경우 어떻게 구현을 해야될까?

만약 클래스 내부에 구현을 한다면 Repairable한 Unit의 개수만큼 오버로딩된 메소드가 구현되어야 할 것이다. 조상 또한 다르기 떄문에 최대한 공통적으로 묶는다 한들 결국 2개 이상의 메소드에 대한 구현이 불가피한 것이다. 

이런 경우 Repairable 인터페이스를 연결해줌으로써 해결이 가능하다.

```java
class Test{
	public static void main(String[] args){
		Tank tank = new Tank();
		SCV scv = new SCV();
		scv.repair(tank);
	}
}

class SCV extends GroundUnit implements Repairable{
	SCV(){
		// aa
	}

	void repair(Repairable r){
		if(r instanceof Unit){
			// aa
		}
	}
}
```

r이 Repairable 타입이기 때문에 인터페이스에 정의된 멤버만이 사용가능하다. 이 점을 활용하여 instanceof로 r이 Unit 중의 하나인지를 검증한 후에 구현코드를 작성하면 된다.

또 다른 점에서 인터페이스가 편리한것이 클래스 A와 B가 A->B의 형태를 가지고 있을 경우에 한 쪽에 대한 내용을 수정할 경우 두 클래스 내부의 관련된 코드를 모두 수정해야 된다는 특징이 있다. 이런 부분에서 인터페이스를 통해 접근을 하게 되면 둘 중 하나에서 변경이 필요한 경우 나머지 하나를 변경하지 않고 인터페이스 내부의 코드만을 변경하면 된다는 장점이 있다. 즉 A->I->B의 형태가 되는 것이다.

## Settings

개발을 진행하기 전에 안드로이드 관련 기초적인 내용들에 대한 많은 공부를 하였다. 안드로이드 역시 Django와 비슷하게 꽤 자세한 document가 있다. 

[안드로이드 개발자 문서](https://developer.android.com/guide/components/intents-filters?hl=ko)

공부한 내용들에 대한 키워드를 간략히 남겨보자면 안드로이드 레이아웃에 대한 개념, 라이프 사이클, 4가지 구성 요소, 액티비티 스택, 프레그먼트 등이 있었다. 

개발의 순서에 대한 설명은 아래의 링크를 참고하기로 하였다.

[어플리케이션 제작 순서](https://devcompass.co.kr/%EC%95%B1-%EC%A0%9C%EC%9E%91/)

### AndroidStudio

안드로이드 개발은 안드로이드 스튜디오를 사용하는 것이 가장 편리해보인다. qemu 에뮬레이터도 있을 뿐 아니라 pycharm과 거의 동일한 IDE 기능을 가지고 있다. 거기다가 layout을 쉽게 그릴 수 있도록 도와주니 더욱 좋다. 

안드로이드 개발을 위해서는 먼저 java를 사용하기 위한 jdk를 설치해야 되며 그 후 안드로이드 스튜디오를 설치하면 dependency가 대부분 설치가 된다. 그리고 이는 brew를 사용하면 편리하게 설치가 가능하다.

```console
brew cask install android-studio
```

### Docker로 graphql-django 생성하기

이번에는 Django를 graphql과 엮어보기로 했다. 이를 위해서 Dockerfile을 아래와 같이 작성하였다.

```console
FROM python:3

MAINTAINER Wizley <wizley@kakao.com>

WORKDIR /code
COPY ./requirements.txt /code

RUN pip3 install -r ./requirements.txt
```

그리고 requirements.txt에는 graphql 관련 라이브러리가 추가되었다.

```console
aniso8601==7.0.0
asgiref==3.2.3
attrs==19.3.0
coverage==5.0.3
Django==3.0.3
django-cors-headers==3.2.1
graphene==2.1.8
graphene-django==2.9.1
graphql-core==2.3.1
graphql-relay==2.0.1
more-itertools==8.2.0
packaging==20.1
pluggy==0.13.1
promise==2.3
py==1.8.1
PyJWT==1.7.1
pyparsing==2.4.6
pytest==5.3.5
pytest-cov==2.8.1
pytz==2019.3
Rx==1.6.1
singledispatch==3.4.0.3
six==1.14.0
sqlparse==0.3.0
wcwidth==0.1.8
```

직접 설치를 하게 된다면 아래와 같은 명령어로 설치를 진행하면 된다.

```console
pip install graphene-django
```

이제 도커 이미지를 생성할 차례이다. graphql-django라는 이름으로 생성하기로 하였다. 안드로이드를 자바로 짜기 때문에 Spring에 붙히면 어떨까라는 고민을 잠깐 했었지만 이번 프로젝트의 몸통은 안드로이드 개발에 두고 있기 때문에 전에 개발에 사용했던 Django를 사용하기로 했다.

```console
Wizley:~/git/aintstagram/backend_api # docker build --tag graphql-django:1.0 .
```

명령어를 통해 확인을 해보니 잘 설치 된 것을 확인 가능하다.

```console
Wizley:~/git/aintstagram/backend_api # docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
graphql-django      1.0                 a5fe92f02b0f        6 seconds ago       984MB
django              1.0                 0472bb664767        6 weeks ago         981MB
ubuntu              latest              72300a873c2c        7 weeks ago         64.2MB
phpmysqli           1.0                 b5cd17f37d99        7 weeks ago         83.9MB
python              3                   efdecc2e377a        2 months ago        933MB
nginx               1.17                2073e0bcb60e        2 months ago        127MB
mysql               8.0.19              791b6e40940c        2 months ago        465MB
```

이제 컴포즈를 올려서 실행을 시켜보도록 하겠다. 

```
Wizley:~/git/aintstagram/backend_api # cat docker-compose.yml
version: '3'

services:
 graphql:
  container_name: aintstagram
  image: graphql-django:1.0
  environment:
    - DJANGO_DEBUG=true
  volumes:
    - ./code:/code
  ports:
    - "8000:8000"
  tty: true
  command: python3 ./backend_aintstagram/manage.py runserver 0:8000
```

정상적으로 잘 올라간다. Settings.py에서 SECRET_KEY만 슬쩍 빼주면 아주 기본적인 세팅은 끝이 난 것 같다. 

이제 graphql이 제대로 작동하는지 테스트를 위해 아래와 같이 좋은 링크 두개를 찾을 수 있었다.

[Integrate GraphQL into your Django project](https://medium.com/@zoejoyuliao/django-graphql-react-1-integrate-graphql-into-your-django-project-ff51237bb5d9)

[Python으로 GraphQL 서버 구현](https://jonnung.dev/graphql/2019/08/05/python-graphql-graphene-tutorial/)

```python
INSTALLED_APPS = [
    ...
    'graphene_django',
    'api',
]
```

api라는 app을 하나 생성하고 INSTALLED_APPS에 추가해주었따. 그 후 메인의 url을 통해 graphql에 접근할 수 있도록 링크를 하나 설정해주었다. 
(이 경우에 csrf_exempt를 설정해주지 않으면 지금 상황에서는 400에러가 뜨게 된다.)

```python
from django.contrib import admin
from django.urls import path
from django.views.decorators.csrf import csrf_exempt
from graphene_django.views import GraphQLView

urlpatterns = [
    path('admin/', admin.site.urls),
    path('graphql/', csrf_exempt(GraphQLView.as_view(graphiql=True))),
]
```

그 후 test에 사용할 UserModel을 api/models.py에 생성한다.

```python
from django.db import models


class UserModel(models.Model):
    name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)
```

그리고 통신을 위해 사용할 부분을 api/schema.py에 추가해준다.

```python
import graphene
from graphene_django import DjangoObjectType

from api.models import UserModel


class UserType(DjangoObjectType):
    class Meta:
        model = UserModel


class Query(graphene.ObjectType):
    users = graphene.List(UserType)

    def resolve_users(self, info):
        return UserModel.objects.all()


schema = graphene.Schema(query=Query)
```

마지막으로 schema를 settings.py에 추가해주면 된다.

```python
# Graphql

GRAPHENE ={
    'SCHEMA': 'api.schema.schema'
}
```

이제 localhost:8000/graphql/ 링크로 접속하면 아래와 같은 페이지를 확인할 수 있다.


![graphql_init](https://raw.githubusercontent.com/wizleysw/wizleysw.github.io/master/_posts/img/aintstagram/graphql_init.png)

이렇게 graphql을 사용하기 위한 환경구성을 마무리하였다!


### Layout 만들기

안드로이드의 구조에서 res 내부의 Layout은 Django의 template과 같은 기능을 한다. xml로 설계된 button, text 등에 대한 요소들을 java 파일에서 가져와 사용하는 방식으로 구현된다. 

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}
```

기본 코드의 MainActivity를 보면 onCreate 부분에서 ContentView로 layout을 가져와서 뿌려준다. 이를 통해 activity_main.xml 파일의 내용이 해석되 렌더링 된다.

layout을 만들기 위해 인스타그램 어플이 가지고 있는 여러 페이지들에 대한 디자인을 따오는 작업이 필요하다. 즉 화면 정의서를 만들어야 되는데 이는 화면의 구조와 표시될 내용 및 기능에 대한 설명을 적어놓은 와이어 프레임을 작성해야 된다. 역시나 아주 좋은 프로그램이 존재했다. 바로 카카오에서 만든 Oven이다.

[Oven](https://ovenapp.io/)

웹에서 간단하게 와이어 프레임을 작성가능하다. 이제 해야될 일은 레이아웃을 따보는 것이다. 클론 프로젝트의 장점은 잘 만들어진 어플을 디자인 구현에 대한 적은 고민으로 구현해볼 수 있다는 점이다. 아마 혼자 디자인을 한다면 올해안에 끝을 내지 못할수도 있을 것 같다. 

[인스타 메인 페이지 와이어 프레임](https://ovenapp.io/project/wBMk8RazGBK2TMSnzpL0Ah5MpadQfXWU#1wazS)

대충 모양에 대해서 흉내를 내보았다. 이런 식으로 골격과 사이즈를 재서 디자인을 진행하는 것 같다.

## Android Side 개발

이제 하나씩 시작을 해보도록 하자. 프로그램의 전체적인 흐름을 대략적으로 살펴보았다. 테스트를 위해서는 백엔드쪽 DB에 대한 설계가 좀 필요할 것 같긴한데 프로그램이 시작되면서 흐르는 루틴에 따라 개발을 해보기로 하였다.

### IntroActivity

많은 어플리케이션이 실행이 되면 로고를 화면의 중간에 몇초간 띄웠다가 MainActivity를 띄우게 된다. 이를 위한 작업을 IntroActivity로 보통 명명한다. 

[안드로이드 intro 화면 만들기](https://boheeee.tistory.com/4)

위의 사이트에 IntroActivity를 구현한 내용이 있다. 위의 저자의 경우 LinearLayout을 사용하여 구현하였는데 나의 경우 ConstraintLayout을 사용하여 구현하였다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#000000"
    tools:context=".IntroActivity">

    <androidx.constraintlayout.widget.Guideline
        android:id="@+id/guideline_top"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toTopOf="@id/logo"
        app:layout_constraintGuide_begin="100dp"/>

    <androidx.constraintlayout.widget.Guideline
        android:id="@+id/guideline_bottom"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        app:layout_constraintTop_toBottomOf="@id/logo"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintGuide_end="100dp"/>

    <androidx.constraintlayout.widget.Guideline
        android:id="@+id/guideline_left"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        app:layout_constraintTop_toBottomOf="@id/guideline_top"
        app:layout_constraintBottom_toTopOf="@id/guideline_bottom"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toLeftOf="@id/logo"
        app:layout_constraintGuide_begin="30dp"/>


    <androidx.constraintlayout.widget.Guideline
        android:id="@+id/guideline_right"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        app:layout_constraintTop_toBottomOf="@id/guideline_top"
        app:layout_constraintBottom_toTopOf="@id/guideline_bottom"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintLeft_toRightOf="@id/logo"
        app:layout_constraintGuide_end="30dp"/>

    <ImageView
        android:id="@+id/logo"
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:src="@drawable/aintstagram"
        app:layout_constraintBottom_toTopOf="@id/guideline_bottom"
        app:layout_constraintTop_toBottomOf="@id/guideline_top"
        app:layout_constraintLeft_toRightOf="@id/guideline_left"
        app:layout_constraintRight_toLeftOf="@id/guideline_right"/>


</androidx.constraintlayout.widget.ConstraintLayout>
```

Guideline을 활용하여 레이아웃을 짜보았는데 위, 아래, 왼쪽, 오른쪽에 해당하는 가이드라인을 통해 중간부분에 위치하도록 하였다. 그리고 비율에 따라 같은 위치에 존재하도록 dp로 width와 height를 주었다.

```xml
<activity android:name=".IntroActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
<activity android:name=".MainActivity">
</activity>
```

manifest 부분에서 MainActivity가 가지고 있던 intent filter를 IntroActivity로 옮겨주었고 나머지 내용은 블로그와 동일하게 작성하였다. 이를 통해 어플이 실행되면 IntroActivity가 호출이 되고 쓰레드가 호출되어 handler의 결과에 따라 MainActivity로 이동하도록 하였다.

아, 참! 어플의 이름은 aintstagram인데 aint + instagram의 합성어이다. 말 그대로 가짜 인스타그램이라는 의미를 지니고 있다. 그래서 로고는 살짝 모양을 변형하여 대충 쓱 그려보았다. 이를 통해 실행되는 IntroActivity의 모습은 다음과 같다.

![introactivity](https://raw.githubusercontent.com/wizleysw/wizleysw.github.io/master/_posts/img/aintstagram/introactivity.png)

### 런처 아이콘 변경

기본 어플리케이션의 아이콘이 설정되어 있지 않은 상태이기 때문에 이를 설정하기 위해 아래의 블로그를 참고하였다.

[Android 런처 아이콘 변경하기](https://webnautes.tistory.com/1306)

Image Asset을 추가한 뒤 manifest를 수정해주면 된다.

```xml
<application
    android:allowBackup="true"
    android:icon="@mipmap/ic_launcher_aintstagram"
    android:label="@string/app_name"
    android:roundIcon="@mipmap/ic_launcher_aintstagram_round"
    android:supportsRtl="true"
    android:theme="@style/AppTheme">
```

xml파일이 ic_launcher_aintstagram이기 때문에 icon과 roundIcon을 다음과 같이 변경해주면 아래와 같이 변경되는 것을 확인할 수 있다.

![icon](https://raw.githubusercontent.com/wizleysw/wizleysw.github.io/master/_posts/img/aintstagram/icon.png)

### 액션바 숨기기

Empty Activity를 생성해도 윗 부분에 액션바가 존재한다. 실제 어플에서는 해당 부분이 존재하지 않기 때문에 이를 지워주고 싶었다. 방법은 간단했다. manifest에서 theme을 다음과 같이 변경해주면 된다.

```xml
android:theme="@style/Theme.AppCompat.Light.NoActionBar">
```

### 카메라 인텐트 구현하기 

메인 엑티비티로 칭하는 페이지는 인스타그램의 로딩이 완료되면 나오는 창이다. 레이아웃을 짜야했는데 어떤 식으로 짜야될지에 대해서 정말 많은 고민을 하였다. 일단은 직관적인 방식으로 진행하기로 하였고 constraint layout을 활용하여 버튼에 대한 뷰를 만들었다. 그리고 대충 그린 이미지로 ImageButton을 채워서 뷰를 이렇게 구성해주었다.

![main_1](https://raw.githubusercontent.com/wizleysw/wizleysw.github.io/master/_posts/img/aintstagram/main_1.png)

이제 버튼이 동작을 하도록 구현할 것인데 Intent를 활용하여 좌측 상단의 카메라 버튼을 클릭하면 카메라 창이 뜨도록 할 것이다. 이를 위해서는 여러가지 설정이 필요하다. 일단 만든 레이아웃은 다음과 같다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.constraintlayout.widget.Guideline
        android:id="@+id/guideline_top_menu"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintGuide_begin="60dp"/>

    <androidx.constraintlayout.widget.Guideline
        android:id="@+id/guideline_bottom_menu"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintGuide_end="60dp"/>

    <ImageButton
        android:id="@+id/button_to_camera"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintVertical_weight="1"
        app:layout_constraintHorizontal_weight="1"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toTopOf="@id/guideline_top_menu"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toLeftOf="@id/button_logo"
        android:src="@drawable/camera"
        android:scaleType="fitCenter"
        android:background="@android:color/background_dark"
        tools:ignore="MissingConstraints" />

    <Button
        android:id="@+id/button_logo"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintVertical_weight="3"
        app:layout_constraintHorizontal_weight="3"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toTopOf="@id/guideline_top_menu"
        app:layout_constraintLeft_toRightOf="@id/button_to_camera"
        app:layout_constraintRight_toLeftOf="@id/button_to_chat"
        android:text="Aintstagram"
        android:textColor="@android:color/white"
        android:textStyle="italic"
        android:background="@android:color/background_dark"
        tools:ignore="MissingConstraints" />

    <ImageButton
        android:id="@+id/button_to_chat"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintVertical_weight="1"
        app:layout_constraintHorizontal_weight="1"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toTopOf="@id/guideline_top_menu"
        app:layout_constraintLeft_toRightOf="@id/button_logo"
        app:layout_constraintRight_toRightOf="parent"
        android:src="@drawable/sms"
        android:scaleType="fitCenter"
        android:background="@android:color/background_dark"
        tools:ignore="MissingConstraints" />

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/story"
        android:layout_height="100dp"
        android:layout_width="match_parent"
        app:layout_constraintTop_toTopOf="@id/guideline_top_menu"
        app:layout_constraintBottom_toTopOf="@id/scroll"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        android:scrollbars="horizontal"
        tools:ignore="MissingConstraints" />

    <ImageButton
        android:id="@+id/button_to_home"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintVertical_weight="1"
        app:layout_constraintHorizontal_weight="1"
        app:layout_constraintTop_toBottomOf="@id/guideline_bottom_menu"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toLeftOf="@id/button_to_search"
        android:src="@drawable/home"
        android:scaleType="fitCenter"
        android:background="@android:color/background_dark"
        tools:ignore="MissingConstraints" />

    <ImageButton
        android:id="@+id/button_to_search"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintVertical_weight="1"
        app:layout_constraintHorizontal_weight="1"
        app:layout_constraintTop_toBottomOf="@id/guideline_bottom_menu"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toRightOf="@id/button_to_home"
        app:layout_constraintRight_toLeftOf="@id/button_to_add"
        android:src="@drawable/search"
        android:scaleType="fitCenter"
        android:background="@android:color/background_dark"
        tools:ignore="MissingConstraints" />

    <ImageButton
        android:id="@+id/button_to_add"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintVertical_weight="1"
        app:layout_constraintHorizontal_weight="1"
        app:layout_constraintTop_toBottomOf="@id/guideline_bottom_menu"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toRightOf="@id/button_to_search"
        app:layout_constraintRight_toLeftOf="@id/button_to_history"
        android:src="@drawable/add"
        android:scaleType="fitCenter"
        android:background="@android:color/background_dark"
        tools:ignore="MissingConstraints" />

    <ImageButton
        android:id="@+id/button_to_history"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintVertical_weight="1"
        app:layout_constraintHorizontal_weight="1"
        app:layout_constraintTop_toBottomOf="@id/guideline_bottom_menu"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toRightOf="@id/button_to_add"
        app:layout_constraintRight_toLeftOf="@id/button_to_info"
        android:src="@drawable/heart"
        android:scaleType="fitCenter"
        android:background="@android:color/background_dark"
        tools:ignore="MissingConstraints" />

    <ImageButton
        android:id="@+id/button_to_info"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintVertical_weight="1"
        app:layout_constraintHorizontal_weight="1"
        app:layout_constraintTop_toBottomOf="@id/guideline_bottom_menu"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toRightOf="@id/button_to_history"
        app:layout_constraintRight_toRightOf="parent"
        android:src="@drawable/userinfo"
        android:scaleType="fitCenter"
        android:background="@android:color/background_dark"
        tools:ignore="MissingConstraints" />

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/scroll"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintTop_toBottomOf="@id/story"
        app:layout_constraintBottom_toBottomOf="@id/guideline_bottom_menu"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        android:scrollbars="vertical"/>

</androidx.constraintlayout.widget.ConstraintLayout>
```

아직 수정의 필요성이 보이긴 하지만 ImageButton과 RecyclerView를 활용하여 constraint Layout을 짜보았다. 카메라 기능을 활용하기 위해서는 Manifest에 아래의 조건을 추가해주어야 된다.

```xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-feature android:name="android.hardware.camera" android:required="true" />
```

이제 MainActivity에서 버튼에 대한 부분을 처리해주어야 되는데 Oncreate부분에 이를 작성할 것이다. 

```java

public class MainActivity extends AppCompatActivity{
    private static final int REQUEST_IMAGE_CAPTRUE = 1;
    private ImageButton btn_camera;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        this.setBtn();

    }

    public void setBtn(){
        btn_camera = (ImageButton)findViewById(R.id.button_to_camera);

        View.OnClickListener Listener = new View.OnClickListener(){
            @Override
            public void onClick(View v) {
                switch (v.getId()) {
                    case R.id.button_to_camera:
                        int permissionCheck = ContextCompat.checkSelfPermission(MainActivity.this, Manifest.permission.CAMERA);
                        if(permissionCheck == PackageManager.PERMISSION_DENIED){
                            ActivityCompat.requestPermissions(MainActivity.this, new String[]{Manifest.permission.CAMERA},0);
                        } else {
                            Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
                            if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
                                startActivityForResult(takePictureIntent, REQUEST_IMAGE_CAPTRUE);
                            }
                        }
                        break;
                    case R.id.button_to_home:
                        break;
                }
            }
        };

        btn_camera.setOnClickListener(Listener);
    }
}
```

setBtn이라는 함수를 생성하였는데 이 함수는 버튼들에 대한 초기화를 진행할 것이다. 함수가 호출이 되면 btn_camera라는 ImageButton이 생성되는데 이 버튼에 OnClickListener를 달아줄 것이다. OnclickListener를 정의해줄 Listener라는 변수를 생성하는데 이 변수는 내부에서 onClick을 오버라이딩해서 해당 이벤트 발생시의 동작을 재정의해준다. getId를 통해 어떤 버튼이 클릭 되었는지를 판별한 뒤 해당 버튼에 따라 case문을 수행해주는 것이다. button_to_camera가 onClick되면 takePictureIntent를 실행시켜줄 것인데 해당 인텐트는 카메라 작업을 위한 사전정의된 인텐트이다. 하지만 안드로이드 특정 버전 이상부터는 해당 기능에 대한 퍼미션 검사를 해주지 않으면 Permission 관련 에러가 발생하다. 그래서 이를 체크하기 위해서 requestPermission을 위와같이 진행해주어야 된다. 이렇게 코드를 작성하면 처음 카메라 버튼이 클릭되면 권한에 대한 요청이 수행되고 그 후에는 카메라 기능이 정상적으로 작동하게 된다. 

자 이번에는 받아서 처리하는 부분을 설계해야되는데 Intent에 대한 결과를 받아오는 onActivityResult을 오버라이딩 해서 구현해 주어야 된다.

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data){
    super.onActivityResult(requestCode, resultCode, data);
    if(requestCode == REQUEST_IMAGE_CAPTRUE && resultCode == RESULT_OK && data.hasExtra("data")) {
        Bitmap bitmap = (Bitmap) data.getExtras().get("data");
    }
}
```

여기서 엄청난 시간을 낭비하였는데 startActivityForResult로 넘겨주는 인터넷 상의 많은 코드들이 bitmap을 가져오는 getExtras()에서 null 문제가 발생하였다. 삽질한 결과 activity를 넘겨주기 전에 putExtra로 URI 정보를 넘겨줄 경우에 이런 문제가 발생하였다. 후.. 어쨌든 지금 결과를 받아오는 곳에서 bitmap으로 찍은 사진에 대한 정보를 가져오는 것 까지 확인할 수 있었다. 또 그 과정에서 adb를 통해서 qemu내부에 접속을 시도하였는데 root권한을 획득하는 것이 안되었다. 이것도 qemu 안드 에뮬레이터 중에서 Google Play와 연관된 이미지를 사용하면 안되는 것이었다. 이를 위해서 다른 버전의 에뮬레이터를 추가로 설치하고서야 adb로 접속하여 내부 구조를 확인할 수 있었다. adb의 경우 android-studio의 sdk가 설치된 파일로 가서 platform-tools 폴더 내부에 설치되어있어서 따로 설치를 할 필요가 없다. 

### 갤러리 인텐트 구현하기

카메라와 비슷한 방식으로 구현이 가능하다.

```java
View.OnClickListener Listener = new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        int permissionCheck;
        switch (v.getId()) {
            case R.id.button_to_camera:
                permissionCheck = ContextCompat.checkSelfPermission(MainActivity.this, Manifest.permission.CAMERA);
                if (permissionCheck == PackageManager.PERMISSION_DENIED) {
                    ActivityCompat.requestPermissions(MainActivity.this, PERMISSIONS, 0);
                } else {
                    Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
                    if (intent.resolveActivity(getPackageManager()) != null) {
                        startActivityForResult(intent, REQUEST_IMAGE_CAPTRUE);
                    }
                }
                break;

            case R.id.button_to_add:
                permissionCheck = ContextCompat.checkSelfPermission(MainActivity.this, Manifest.permission.READ_EXTERNAL_STORAGE);
                if (permissionCheck == PackageManager.PERMISSION_DENIED) {
                    ActivityCompat.requestPermissions(MainActivity.this, PERMISSIONS, 0);
                } else {
                    Intent intent = new Intent();
                    intent.setAction(Intent.ACTION_GET_CONTENT);
                    intent.setData(MediaStore.Images.Media.EXTERNAL_CONTENT_URI);
                    intent.setType("image/*");
                    startActivityForResult(intent, REQUEST_TAKE_ALBUM);
                }
                break;
        }
    }
};

```

마찬가지로 정보를 가져오는 것이기 때문에 그에 상응하는 권한을 Manifest에 정의를 해주어야 되며 User로 부터 동의를 구해야 된다. 

```xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
```

이렇게 해주면 갤러리로 부터 사진을 가져오는 것이 가능하다.

![gallery](https://raw.githubusercontent.com/wizleysw/wizleysw.github.io/master/_posts/img/aintstagram/gallery.png)

### 카카오 로그인 구현

[카카오 계정으로 로그인하기](https://re-build.tistory.com/9)

사실 가장 기본적인 로직상 IntroActivity가 실행되고 나서 login여부에 따라 MainActivity로 메인 페이지로 이동시킬지 LoginActivity로 로그인 관련 로직을 처리할지를 결정지어야 된다. 현재의 흐름은 IntroActivity가 LoginActivity가 아닌 MainActivity로 바로 이동하기 때문에 이 부분에 대한 구현이 진행되어야 한다.

이번에는 카카오 api를 사용해서 로그인을 구현해보기로 하였다. 카카오 로그인을 사용하면 좋은 점이 여러가지 정보들에 대한 처리를 나의 서버가 아닌 카카오쪽에 전담할 수 있다. 계정에 대한 유출에 대한 고민을 줄일 수 있고 관련 DB에 처리를 부담하지 않을 수 있기 때문에 익숙해진다면 개발의 속도를 늘릴 수 있다. 

이제 API를 사용해볼건데 이를 위해서는 Android용 SDK를 설치하여야 한다. 카카오 디벨로퍼에서 해당 sdk를 다운받거나 Gradle에 추가하는 방향으로 진행할 수 있다. Gradle은 프로젝트의 빌드에 사용되며 Maven을 통해 써드파티 라이브러리를 추가할 수 있다.

[카카오 SDK 적용 공식문서](https://developers.kakao.com/docs/latest/ko/getting-started/sdk-android-v1)

aintstagram/build/build.gradle 내부에 아래와 같이 라이브러리를 추가해준다.

```javascript
subprojects {
    repositories {
        google()
        jcenter()
        maven { url 'https://devrepo.kakao.com/nexus/content/groups/public/' }
    }
}
```

그리고 app 아래의 build gradle에 implementation을 추가해준다.

```javascript
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])

    implementation 'androidx.appcompat:appcompat:1.1.0'
    implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
    implementation 'androidx.recyclerview:recyclerview:1.1.0'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'androidx.test.ext:junit:1.1.1'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.2.0'

    implementation group: 'com.kakao.sdk', name: 'usermgmt', version: '1.27.0'
    // 카카오톡
    implementation group: 'com.kakao.sdk', name: 'kakaotalk', version: '1.27.0'
    // 카카오링크
    implementation group: 'com.kakao.sdk', name: 'kakaolink', version: '1.27.0'
}
```

이제 Manifest에서 API에 대한 네이티브 키를 등록해주면 된다.

```xml
<meta-data
    android:name="com.kakao.sdk.AppKey"
    android:value="{NATIVE_APP_KEY}" />
```

해시에 대한 정보를 아래의 명령어를 입력하여 생성 후에 kakao.developer.com에 등록해준다.

```console
Wizley:~/git/aintstagram # keytool -exportcert -alias androiddebugkey -keystore ~/.android/debug.keystore -storepass android -keypass android | openssl sha1 -binary | openssl base64
```

그리고 테스트를 위해서 빌드를 시키면 정상적으로 완료되는 것을 확인할 수 있다.

[카카오 로그인 가이드](https://developers.kakao.com/docs/latest/ko/kakaologin/common)

카카오의 로그인 서비스는 REST api를 통해 구현되며 토큰을 주고 받는 방식으로 진행된다고 한다. 로직을 보면 사용자가 정보를 요청하면 토큰을 발급해서 넘겨주는 방식인 것 같다. LoginActivity를 위한 레이아웃을 설계하였다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:background="#000000"
    tools:context=".LoginActivity">>

    <androidx.constraintlayout.widget.Guideline
        android:id="@+id/guideline_top"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toTopOf="@id/logo"
        app:layout_constraintGuide_begin="100dp"/>

    <androidx.constraintlayout.widget.Guideline
        android:id="@+id/guideline_bottom"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        app:layout_constraintTop_toBottomOf="@id/logo"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintGuide_end="100dp"/>

    <androidx.constraintlayout.widget.Guideline
        android:id="@+id/guideline_left"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        app:layout_constraintTop_toBottomOf="@id/guideline_top"
        app:layout_constraintBottom_toTopOf="@id/guideline_bottom"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toLeftOf="@id/logo"
        app:layout_constraintGuide_begin="30dp"/>


    <androidx.constraintlayout.widget.Guideline
        android:id="@+id/guideline_right"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        app:layout_constraintTop_toBottomOf="@id/guideline_top"
        app:layout_constraintBottom_toTopOf="@id/guideline_bottom"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintLeft_toRightOf="@id/logo"
        app:layout_constraintGuide_end="30dp"/>

    <ImageView
        android:id="@+id/logo"
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:src="@drawable/aintstagram"
        app:layout_constraintBottom_toTopOf="@id/guideline_bottom"
        app:layout_constraintTop_toBottomOf="@id/guideline_top"
        app:layout_constraintLeft_toRightOf="@id/guideline_left"
        app:layout_constraintRight_toLeftOf="@id/guideline_right"/>

    <com.kakao.usermgmt.LoginButton
        android:id="@+id/login_button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="20dp"
        android:layout_marginRight="20dp"
        android:layout_marginBottom="30dp"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintBottom_toBottomOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

기존의 introActivity에 카카오에서 제공하는 LoginButton을 추가하였다. 

![login](https://raw.githubusercontent.com/wizleysw/wizleysw.github.io/master/_posts/img/aintstagram/loginactivity.png)

이를 핸들링하기 위한 LoginActivity.java 파일은 아래와 같다.

```java
package com.ssg.aintstagram;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.util.Log;
import androidx.annotation.Nullable;
import com.kakao.auth.ISessionCallback;
import com.kakao.auth.Session;
import com.kakao.util.exception.KakaoException;

public class LoginActivity extends Activity {

    private ISessionCallback sessionCallback = new ISessionCallback() {
        @Override
        public void onSessionOpened() {
            Intent intent = new Intent(LoginActivity.this, MainActivity.class);
            startActivity(intent);
        }

        @Override
        public void onSessionOpenFailed(KakaoException exception) {
            
        }
    };

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);
        
        Session.getCurrentSession().addCallback(sessionCallback);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();

        Session.getCurrentSession().removeCallback(sessionCallback);
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);

        if(Session.getCurrentSession().handleActivityResult(requestCode, resultCode, data)){
            return;
        }

        super.onActivityResult(requestCode, resultCode, data);
    }
}
```

해당 액티비티가 생성이 되면 activity_login으로 view가 생성이 되고 getCurrentSession()을 통해 세션에 대한 정보를 가져온다. 만약 존재하면 MainActivity를 호출하게 된다. 여기서 이를 처리하기 위한 sdk에 대한 코드를 생성해야 된다.

```java
package com.ssg.aintstagram;

import android.app.Application;
import android.content.Context;
import com.kakao.auth.IApplicationConfig;
import com.kakao.auth.KakaoAdapter;
import com.kakao.auth.KakaoSDK;

public class MyKakaoAdapter extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        // SDK 초기화
        KakaoSDK.init(new KakaoAdapter() {

            @Override
            public IApplicationConfig getApplicationConfig() {
                return new IApplicationConfig() {
                    @Override
                    public Context getApplicationContext() {
                        return MyKakaoAdapter.this;
                    }
                };
            }
        });
    }
}
```

MyKakaoAdapter라는 KakaoAdapter를 application으로 생성해주면 된다. 이를 위해선 Manifest에 추가해주어야 된다.

```xml
<application
    android:name=".MyKakaoAdapter"
```

이제 테스트를 해보면 아이디/패스워드로 요청을 보내고 사용자의 동의를 얻으면 MainActivity로 이동하는 것을 확인할 수 있다.

```xml
generic_x86_arm:/data/data/com.ssg.aintstagram/shared_prefs # ls
3f5c6005c0954ba8a11e315b702a5442.xml  WebViewChromiumPrefs.xml
at 3f5c6005c0954ba8a11e315b702a5442.xml                                                                      
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
    <string name="com.kakao.token.RefreshToken.ExpiresAt">{&quot;value&quot;:1592044787261,&quot;valueType&quot;:&quot;long&quot;}</string>
    <string name="com.kakao.token.AccessToken.ExpiresAt">{&quot;value&quot;:1586903987261,&quot;valueType&quot;:&quot;long&quot;}</string>
    <string name="com.kakao.token.RefreshToken">{&quot;value&quot;:&quot;something&quot;,&quot;valueType&quot;:&quot;string&quot;}</string>
    <string name="com.kakao.token.AccessToken">{&quot;value&quot;:&quot;something&quot;,&quot;valueType&quot;:&quot;string&quot;}</string>
</map>
```

adb로 접속해서 확인해보면 Token에 대한 정보가 로컬 저장소에 정상적으로 저장된 것을 확인할 수 있었다. 여기서 조금 더 나아가 수정을 해보자. 현재까지 있던 문제로는 introActivity가 activityStack에 남아있다는 점과 MainActivity로 넘어간다는 점, 또 그리고 LoginActivity를 세션이 유지되어 있는 동안에는 굳이 띄울필요가 없다는 점을 들 수 있겠다. 해당 처리를 위해서 IntroActivity 부분과 IntroThread를 낭낭히 수정해보자.

```java
package com.ssg.aintstagram;

import android.os.Message;
import android.os.Handler;
import com.kakao.auth.Session;

public class IntroThread extends Thread {
    private Handler handler;

    public IntroThread(Handler handler){
        this.handler = handler;
    }

    @Override
    public void run(){
        Message msg = new Message();

        try{
            Thread.sleep(2000);

            if(Session.getCurrentSession().isOpened()){
                msg.what = 1;
            }
            else{
                msg.what = 2;
            }
            handler.sendEmptyMessage(msg.what);
        } catch (Exception e){
            e.printStackTrace();
        }
    }
}
```

Introthread는 잠시 로고를 띄워주는 역할을 수행했었는데 한 가지 작업을 더 추가하였다.

[kakao-api doc](https://developers.kakao.com/sdk/reference/android-legacy/release/index.html)

위의 사이트의 Session 라이브러리를 보면 isOpened라는 메소드가 있다. 해당 메소드를 호출하여 세션에 대한 정보가 있는지를 확인한 뒤 연결되어 있는 상태이면 1을 아닐 경우 2를 넘겨준다.

```java
@Override
public void handleMessage(Message msg){
    if(msg.what == 1){
        Intent intent = new Intent(IntroActivity.this, MainActivity.class);
        startActivity(intent);
    }
    else{
        Intent intent = new Intent(IntroActivity.this, LoginActivity.class);
        startActivity(intent);
    }
    finish();
}
```    

IntroActivity는 해당 결과를 받아와 LoginActivity를 호출할지 MainActivity를 호출할지 결정해주기만 하면 된다. 그 후 onDestroy() 호출을 위해 finish를 해주면 된다. 


























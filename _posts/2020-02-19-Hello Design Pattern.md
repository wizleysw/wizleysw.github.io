---
published: true
layout: single
title : "Hello Design Pattern!"
category : dev
comments: false
author_profile : true
tag : 
  - design_pattern
  - algorithm
toc : true
---

## HelloWorld!

위즐리의 개발 블로그이다. 해보고 싶었던 개발, 해보고 싶었던 개발과 관련된 내용들을 담을 예정이다. 해킹에 관련된 내용을 담을지는 모르겠다. 현재를 기준으로는 개발이 대부분을 차지하지 않나 싶다. 연구한 항목이 많지만 적히게 될 대부분의 내용은 해오던 것이 아닌 해보고 싶었던 것 위주가 될 듯하다. 위즐리의 블로그 시작!

### Design Pattern

2019년도는 C++로 작성된 오픈소스에 대한 Source-Code Auditing에 대부분의 시간을 썼었다. 그러면서 많은 것을 느꼈는데 일단 가장 필요성을 느꼈던 항목이 코드의 패턴을 이해하는 것이었다. 잘 짜여진 프로그램일수록 그 패턴은 정교하며 비슷한 형태로 구현되어 있다. 이는 개발자들이 코드 컨벤션에 맞게 디자인 패턴을 철저하게 지켰기에 가능한 일이라고 생각한다. 또한 이런 점을 들어볼 때 좋은 코드 패턴이란 정의가 되어있다. 각각의 코드는 그 의미에 따라 좋은 디자인 패턴이 존재하고 이는 가독성 및 유지보수가 큰 도움이 된다. 이 글에서는 디자인 패턴에 대한 내용을 담고자 한다. 현재 하고 있는 공부가 좀 많아서 언제 이 포스팅이 끝날지는 모르겠지만 그래도 틈틈히 내용을 업로드할 예정이다.

## 생성 패턴(Creational Pattern)

생성 패턴은 인스턴스를 만드는 절차에 해당하는 패턴이다. C++, Java등의 객체지향언어의 가장 큰 장점은 객체화를 수행하여 같은 행위를 수행하는 값들에 대한 틀을 클래스의 형태로 정의할 수 있다는 것에 있는데 생성 패턴은 이 객체의 생성 및 추가, 표현 방법등에 사용된다. GOF의 디자인 패턴에 의하면 생성패턴은 두 가지 큰 역할을 수행한다. 첫 째, 시스템이 어떤 구체 클래스를 사용하는지에 대한 정보의 캡슐화를 수행한다. 둘 째로는 클래스의 인스턴스들이 어떤 방식으로 생성되고 혼합되는지에 대한 부분을 가려주는 역할을 수행한다. 즉 이 패턴은 어떤 것을 누가 어떻게 언제 생성하는지에 대한 고민을 담고있다.

### 추상 팩토리

추상 팩토리는 다음의 경우에 사용된다고 한다.

1. 객체가 생성/구성/표현되는 방식과 무관하게 시스템을 독립적으로 만드는 경우
2. 여러 제품중에 하나를 선택하는데, 그 제품이 대체가 가능한 경우
3. 제품 객체들이 같이 사용되도록 설계되었는데 외부에서도 제한사항을 지키고 싶은 경우
4. 제품에 대한 클래스 라이브러리를 제공하고 구현이 아닌 인터페이스를 노출하고 싶은 경우

이 경우에 AbstractFactory는 개념적 제품에 대한 객체를 생성하는 인터페이스를 정의하는 것으로 ConcreteFactory가 구체적인 제품에 대한 객체를 생성하는 역할을 한다. AbstractProduct는 개념적 제품 객체의 인터페이스를 정의하며 ConcreteProduct는 팩토리가 생성할 객체에 대한 구체적 정의 및 AbstractProduct가 정의하는 인터페이스에 대한 실질적 구현부분이다. 

![추상 팩토리](https://raw.githubusercontent.com/wizleysw/wizleysw.github.io/master/_posts/img/design_pattern/abstract_factory.png)

위의 그림에서 AbstractFactory를 구현한 ConcreteFactory는 House 객체의 생성을 담당한다. ConcreteFactory_1은 AbstractProduct를 구현한 ConcreteProduct_1과 2중 1에 대한 생성을 담당하며 2를 생성하기 위해서는 ConcreteFactory_2를 사용해야 된다. 즉 Abstract Factory는 직접적인 생성을 담당하지 않고 이를 생성하기 위한 ConcreteFactory 서브 클래스에 위임한다.

이를 통해 추상 팩토리는 객체를 생성하는 부분과 객체에 대한 책임을 지는 부분을 캡슐화하였고 이를 통해 사용자가 생성할 객체의 클래스에 대한 제어가 가능하다. 또한 객체들에 대한 특징을 묶어서 설계함으로써 객체들의 일관성이 보장된다. 하지만 이러한 특징 때문에 새로운 종류의 제품을 등록하고자 할때 일반적 범주에 속해있지 않은 경우 팩토리에 대한 구현을 변경해야 된다. 다시 말해 인터페이스를 수정하여 새로운 객체를 추가할 방법을 정의하고 서브 클래스들에 대한 구현을 변경해야 된다.

이 규칙을 토대로 대략적을 수도 코드를 만들어보면 다음과 같다.

```c++
class AbstractHumanFactory{
	public:
		Human();

		virtual Human* BornHuman(){
			return new Human();
		}
}

class ManFactory : public HumanFactory{
	public:
		ManFactory();

		virtual Human* BornHuman(){
			return new ManHuman();
		}
}

Human* Hospital::CreateHuman(HumanFactory &factory){
	Human* aHuman = factory.BornHuman();
}

{
	Hospital hostpital;
	ManFactory factory;
	hospital.createHuman(factory);
}
```

### 빌더(Builder)

객체를 생성하는 방법과 표현하는 방법을 정의하는 클래스를 분리하여 동일한 절차를 제공할 수 있도록 하는 것이 빌더의 역할이다. 이 패턴은 다음의 경우에 사용된다. 파서를 예로 들면 어떤 텍스트를 읽어와 파싱하는 과정은 매번 동일한데 이를 변환하는 언어는 다양할 수 있다. 이런 경우에 다른 언어를 추가할 경우 변환을 담당하는 부분은 새로운 형태로 추가가 될 수 있다. 이와 같이 이 분할이 가능한 형태일 때 서브클래스를 쪼개어 다른 변환과정을 거치도록 구현을 한다. 변환하는 과정이 빌더가 되는 것이다.

![빌더](https://raw.githubusercontent.com/wizleysw/wizleysw.github.io/master/_posts/img/design_pattern/builder.png)

Builder는 객체의 특정 부분의 생성에 사용되는 추상 인터페이스이며 ConcreteBuilder는 이에 대한 구현 부분을 의미한다. 또한 이 부분에서 빌더가 복합을 수행한다. 그리고 이를 바탕으로 Product 즉, 복합이 완료된 객체가 생성이 된다. 그리고 Director는 Builder 인터페이스를 사용하는 객체를 합성한다. 즉 Director가 요청하는 명세서를 바탕으로 Builder가 부품을 추가하며 사용자는 Builder를 통해 제품을 조회하게 된다. 

이런 구조에 의해 새로운 제품에 대한 정보를 추가할 때 Builder 클래스로부터 상속을 받는 새로운 서브클래스를 정의하면 된다. 또한 이렇게 세분화되는 서브클래스들은 공통점이 없기 때문에 추상화 작업이 생략된다. 

```c++
class SchoolBuilder{
	public:
		virtual void BuildSchool(){}
		virtual void BuildRoom(int room){}

		virtual School* getSchool(){return 0;};
	protected:
		SchoolBuilder();
}

School* Company::CreateSchool(SchoolBuilder &builder){
	builder.BuildSchool();
	builder.BuildRoom(1);
	return builder.getSchool();
}

School* Company::CreateBigSchool(SchoolBuilder &builder){
	builder.BuildSchool();
	builder.BuildRoom(11111111);
	return builder.getSchool();
}

```

이와 같이 구현을 하면 SchoolBuilder의 서브클래스의 구현에 따라 기능을 다르게 구현하는 것이 가능하며 사용자 입장에서는 어떤 방식으로 Room이 생성되는지에 대한 정보를 알 수가 없기 떄문에 캡슐화가 된다고 볼 수 있다. 세부적인 사항을 BigSchoolBuilder와 같이 구현해줌으로써 SchoolBuilder 내부의 메소드들에 대한 세부사항을 구현할 수 있다. 빌더는 추상 팩토리랑 비슷한 기능을 하지만 추상 팩토리는 유사성을 토대로 설계에 집중을 한 반면 빌더는 순차적으로 생성을 마친 뒤 마지막 순간에 객체에 대한 정보를 반환한다는 점이 다르다. 

### 팩토리(Factory)

사용자가 만든 프로그램이 다양한 종류의 문서를 다룰 수 있는 프레임워크를 제공한다고 할 때 Application이라는 클래스와 Document라는 클래스로 크게 추상화를 할 수 있다. 그리고 이 추상화 클래스들은 추상 클래스이기 때문에 그에 따른 구현을 서브클래스에 위임하게 된다. 예를 들어서 text와 관련된 문서일 경우 TextApplication과 TextDocument 클래스 등이 정의 되어야 한다. 여기서 TextApplication은 text와 관련된 Document를 관리하는 역할을, TextDocument는 객체를 생성하는 역할을 주로 담당할 것이다. 이런 경우에 어떤 타입의 문서를 처리하는가에 따라 서브 클래스의 종류가 달라지기 때문에 어떤 종류의 문서를 생성해야 하는지 모르게 된다. 이런 경우에 팩토리 메소드를 사용하여 서브클래스 중 어느 것을 생성해야 되는지에 대한 정보를 캡슐화한다. 그 후 Application의 서브 클래스는 추상화된 메소드를 재정의하여 올바른 Document의 서브 클래스를 그에 맞게 반환하도록 한다. 이런 상황에서 CreateDocument와 같은 연산을 팩토리 메소드라고 하는데 이 의미는 객체를 생성하는 방법을 알고 있다는 의미이다.

팩토리 메서드는 다음의 경우에 사용된다.

1. 클래스가 생성해야되보조 서브클래스는 객체에 대한 클래스를 예측하지 못하는 경우
2. 생성할 객체에 대한 책임을 서브클래스가 지정하도록 하고 싶을 경우
3. 객체의 생성 책임을 보조 서브클래스 중 하나에 위임하고 어떤 서브클래스가 위임자인지에 대한 정보를 모듈화 하고 싶은 경우

![팩토리](https://raw.githubusercontent.com/wizleysw/wizleysw.github.io/master/_posts/img/design_pattern/factory.png)

위 패턴에서 Product는 팩토리 메서드가 생성하게 되는 객체의 인터페이스를 의미하며 ConcreteProduct는 Product의 클래스에 정의된 인터페이스의 실제 구현 부분이다. Creator는 Product 객체를 반환하는 팩토리 메소드이며 여기서는 ConcreteProduct 객체를 반환한다. ConcreteCreator는 팩토리 메소드를 재정의하여 올바른 Product를 가진 ConcreteProduct 인스턴스를 반환해주는 역할을 수행한다.

이렇게 구현하게 되면 특정 클래스가 여러 코드에 종속적이지 않게 구현이 가능하다. 또한 Product에 정의된 인터페이스에 대한 동작만 구현되기에 어떤 ConcreteProduct일지라도 동작을 한다는 장점이 있다. 하지만 이런 구현때문에 하나의 새로운 타입이 생성될 때마다 그에 맞는 Creator 클래스를 서브클래싱해야 된다. 책의 예제를 보면 다음과 같다.

```c++
class MazeGame {
public:
	Maze* CreateMaze();

	virtual Maze* MakeMaze() const{
		return new Maze;
	}
	virtual Room* MakeRoom(int n) const{
		return new Room(n);
	}
}
```

위 코드에서 MakeMaze, MakeRoom이 팩토리 메소드이다. 각각의 메소드는 미로의 구성 요소의 한 파트를 반환한다. 

```c++
Maze* MazeGame::CreateMaze(){
	Maze* aMaze = MakeMaze();

	Room* r1 = MakeRoom(1);
	Room* r2 = MakeRoom(2);
	Door* theDoor = MakeDoor(r1,r2);
}
```

위와 같이 팩토리 메소드를 활용하여 생성이 가능하다. 다른 타입의 미로일 경우 아래와 같이 만들어주면 된다.

```c++
class BombedMazeGame : public MazeGame {
public:
	BombedMazeGame();

	virtual Room* MakeRoom(int n) const{
		return new RoomWithBomb(n);
	}
}
```

기존의 타입을 상속받아 주요 부분들을 재정의 하게 되면 MazeGame의 서브클래스들이 각각의 특성에 의거하여 재정의되어 다양성이 보장된다.

### 원형(Prototype)

원형 패턴은 독립적은 제품을 만들때 사용된다.

1. 인스턴스화할 클래스를 동적 로딩 상황에서 지정할 때
2. 클래스의 인스턴스들이 서로 다른 상태 조합 중에 하나일 경우, 미리 원형으로 초기화 해두고 복제해서 사용하는게 수동적으로 초기화하는 것보다 편리할 때

![프로토타입](https://raw.githubusercontent.com/wizleysw/wizleysw.github.io/master/_posts/img/design_pattern/prototype.png)

Prototype은 자신을 복제하는데 필요한 인터페이스의 정의를 나타내며 ConcretePrototype은 복제와 관련된 메소드를 구현한다. Client는 원형에 복제를 요청하여 새로운 객체의 생성을 수행한다.

이런 특징 덕분에 실행 중에 새로운 제품을 추가/삭제 할 수 있다는 장점이 있다. 또한 객체에 대한 변수값을 지정함으로써 새로운 행위를 하도록 할 수 있다. 즉, 새로운 클래스를 동적으로 정의할 수 있다는 의미이다. 또한 이런 방식으로 인하여 서브클래스의 개수를 줄일 수 있다는 것도 장점이다. 

```c++
class MazePrototypeFactory : public MazeFactory {
public:
	MazePrototypeFactory(Maze*, Room*);

	virtual Maze* MakeMaze() const;
	virtual Room* MakeRoom(int n) const;

private:
	Maze* _prototypeMaze;
	Room* _prototypeRoom;
}
```

위의 Factory는 생성자가 원형을 인자로 받도록 한다.

```c++
MazePrototypeFactory::MazePrototypeFactory(
	Maze* m, Room* r){
	_prototypeMaze = m;
	_prototypeRoom = r;
}
```	

그리고 생성자에서 원형을 기반으로 초기화를 수행한다. 방을 만드는 것 같은 행위들은 원형을 Clone한 뒤 초기화하는 작업을 의미한다.

```c++
Room* MazePrototypeFactory::MakeRoom() const{
	return _prototypeRoom->Clone();
}

Door* MazePrototypeFactory::MakeDoor(Room* r1, Room* r2) const{
	Door* door = _prototypeDoor->Clone();
	door->Initialize(r1,r2);
	return door;
}
```

미로의 종류를 바꾸는 것은 원형을 다른 방식으로 초기화를 하면 된다.

```c++
MazePrototypeFactory bombedMazeFactory(
	new Maze, new BombedWall);
```




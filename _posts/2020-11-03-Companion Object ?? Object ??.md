---
published: true
layout: single
title : "Companion Object ?? Object ??"
category : android
comments: false
author_profile : true
tag : 
  - Android
toc : true
---

## Overview

Kotlin 개발 문서를 읽다가 SingleTon 패턴을 위해서 Object가 사용된다는 사실을 알게 되었는데 여기서 궁금증이 생겼다. 안드로이드 어플을 개발하다보면 Companion Object를 사용하는 예제를 자주 접할 수 있는데 Object와 Companion Object는 대체 무슨 차이가 있는지가 궁금하였다.

### Result

```kotlin
class ObjectTest{
    object JustObject {
        val name = "JustObject"

        fun printName(){
            println("$name")
        }
    }

    companion object CompanionObject{
        val name = "CompanionObject"

        fun printName(){
            println("$name")
        }
    }
}
```

두 타입은 위와같이 동일한 형태로 생성하는 것이 가능하다.

```kotlin
fun main(){
    ObjectTest.JustObject.printName()

    ObjectTest.CompanionObject.printName()
}
```

심지어 Class를 참조하여 호출하는 것도 동일한 방식으로 가능하다.

결론부터 말하자면 아래의 코드에서 차이가 난다

```
ObjectTest.printName()
```

Class에서 바로 메소드를 호출하면 companion object가 호출이 된다. Companion Object는 JAVA의 static과 같이 클래스의 static 변수와 같은 역할을 수행하기 위해 사용된다.
<br>

## 가변성의 단점

가변성은 단점이 많다. 가변성의 단점은 다음과 같다. 

1. 프로그램을 이해하고 디버그 하기가 힘들다. 객체의 상태를 추적해야만 하기 때문이다.
2. 코드의 실행을 추론하기가 어렵다. 상황에 따라 값이 계속 변경되기 때문에 코드의 실행을 예측하기가 힘들다.
3. 멀티스레드 환경에서 동기화가 필요하다.
4. 테스트하기 어렵다. 모든 상태에 대해 테스트가 필요하기 때문이다.
5. 상태의 변경이 일어날 때마다 외부에 이를 알려야 한다. 예를 들어 리스트에 가변 객체가 담겨 있다면, 객체의 상태에 변경이 일어날 때마다 재정렬을 해줘야 한다.
---
# 코틀린에서 가변성 제한하기

## 읽기 전용 프로퍼티 (val)

val 을 사용하면 읽기 전용으로 프로퍼티를 생성할 수 있고, 일반적으로는 한번 할당된 값을 수정할 수 없다. 

```kotlin
val list = listOf(1,2,3)
list.add(4) 
```

```Kotlin
interface Element {  
    val v: Boolean  
}  
  
class ActualElement: Element {  
    override var v: Boolean = false  
}
```

두 번째 방법이 조금 신박하다. override 를 통해서 val 를 var 로 변환하는 방법이다. 하지만 안심하자. 프로퍼티에 대한 레퍼런스는 변경되지 않기 때문에 멀티스레드 환경에서 동시성에 대한 문제를 줄일 수 있다.

여기서 알 수 있듯, `val` 은 `읽기 전용` 프로퍼티지, `불변` 을 의미하는 것은 아니다.
## 가변 컬렉션과 읽기 전용 컬렉션 구분하기

코틀린은 읽고 쓸 수 있는 컬렉션과 읽기 전용 컬렉션으로 구분된다.

![](https://miro.medium.com/v2/resize:fit:786/format:webp/1*ESQLaTW4nDxgMvMeSZk0mg.png)

읽기 전용 컬렉션이 내부의 값을 변경할 수 없는 건 아니지만, 읽기 전용 인터페이스가 이를 지원하지 않기 때문에 변경할 수 없다. 이러한 컬렉션들은 실제로 불변하지 않은 컬렉션을 외부적으로 불견하게 보이게 만들어서 얻어지는 안정성을 얻게 된다. 그래서 다음과 같이 개발자들 의도적으로 불변성을 해치려고 하면 문제가 발생할 수 있다.

```Kotlin
val list = listOf(1,2,3) // return Array.ArrayList (can't add)

if(list is MutableList) // smart casting
	list.add(4)
```

`Array.ArrayList`는 `add`,`set` 과 같은 메서드를 제공하지 않기 때문에 다음과 같은 오류가 발생한다.

```
Exception in thread "main" java.lang.UnsupportedOperationException
	at java.base/java.util.AbstractList.add(AbstractList.java:153)
	at java.base/java.util.AbstractList.add(AbstractList.java:111)
	at org.example.MainKt.main(Main.kt:8)
	at org.example.MainKt.main(Main.kt)
```

이러한 경우 복제를 통해 새로운 `Mutable`컬렉션을 생성해서 사용해야 한다.

```Kotlin
val list = listOf(1,2,3) // return Array.ArrayList (can't add)

val mutableList = list.toMutableList()
mutableList.add(4)
```

### 데이터 클래스의 copy

코틀린에서 제공하는 `data class`는 `immutable`을 보장한다. `data` 한정자는 `copy`라는 메서드를 제공하는데 이를 활용해서 프로퍼티가 같은 새로운 객체를 만들어낼 수 있다. 

---

# 다른 종류의 변경 가능 지점

변경 가능한 리스트를 만들어야 한다고 가정해보면, 다음과 같은 두 가지 선택지가 존재할 것이다.

```Kotlin
val list: MutableList<Int> = mutableListOf()
list += 1 // list.plusAssign(1)

var list: List<Int> = listOf()
list += 1 // list.plus(1)
```

두 가지 선택지는 사용법은 같지만 실제로 동작하는 방식은 다르다. 그리고 또 다른 차이점은 변경 가능 지점의 차이(mutating point)이다. 

첫 번째의 경우, 구체적인 리스트 구현 내부에 변경 가능 지점이 존재한다. 이는 내부적으로 동기화가 잘 되어 있는지 확실하게 알 수 없어서 위험하다.

두 번째의 경우, 프로퍼티 자체가 변경 가능 지점이다.  따라서 멀티스레드 처리의 안정성이 더 좋다고 할 수 있다.

다음과 같이 리스트 변경 사항이 발생하는 경우 변경을 추적하게끔 만들 수도 있다.

```Kotlin
var names by Delegates.observable(listOf<String>()) {_, old, new ->  
    println("Names changed from $old to $new")  
}

names += "a" // Names changed from [] to [a]
names += "b" // Names changed from [a] to [a, b]
```

---
# 변경 가능 지점 노출하지 않기

상태를 나타내는 `mutable` 객체를 외부에 노출하지 말아야 한다. 외부에서 사용하면서 의도적으로 상태를 변경할 수 있기 때문이다.

```Kotlin
fun getUser(): MutableUser = user.copy()
```

위와 같이 `copy`를 해서 복제본을 return 하거나, 복제가 불가능한 상위 타입으로 캐스트해서 리턴하는 방식을 사용해 가변성을 제한할 수도 있다.
### 코틀린의 프로퍼티 위임

코틀린에서는 프로퍼티 위임(Property Delegation)이라는 새로운 기능을 제공한다. 대표적으로 지연 프로퍼티인 `lazy`가 존재한다. 처음 사용하는 요청이 들어올 때 초기화되는 프로퍼티를 의미한다.

```Kotlin
public inline operator fun <T> Lazy<T>.getValue(
	thisRef: Any?, property: KProperty<*>): T = value
```

다음과 같이 Observable 패턴을 쉽게 만들 수 있다.

```Kotlin
var items: List<Item> by
	Delegates.observable(listOf()) { _, _, _ ->
		notifyDataSetChanged()
	}
```
- `Delegates.observable()`은 위임한 프로퍼티를 읽고/쓸 때마다 getValue/setValue를 호출한다.

### 프로퍼티 위임 활용

```Kotlin
var token: String? by LoggingProperty(null)

private class LoggingProperty<T>(var value: T) {
	operator fun getValue(
		thisRef: Any?,
		prop: KProperty<*>
	): T {
		print("${prop.name} returned ${value}")
		return value
	}
	operator fun setValue(
		thisRef: Any?,
		prop: KProperty<*>,
		newValue: T
	): T {
		print("")
		val name = prop.name
		value = newValue
	}
}
```

위와 같은 식으로 새로운 클래스를 만들어서 get/set 할 때마다의 동작을 추가할 수 있다. 위와 같은 프로퍼티 위임이 어떤 식으로 동작하는 지를 알기 위해서는 컴파일 됐을 때의 형태를 보는 것이 좋다.

```Kotlin
@JvmField
private val 'token$delegate' =
	LogginProperty<String?>(null)
var token: String?
	get() = 'token$delegate'.getValue(this, ::token)
	set(value) {
		'token$delegate'.setValie(this, ::token, value)
	}
```

위의 코드를 보면 알 수 있듯 단순하게 값만 처리하는 것이 아니라, 컨텍스트(this)와 프로퍼티 레퍼런스의 경계도 함께 사용하는 형태로 바뀐다.

- 프로퍼티 레퍼런스 : 이름/어노테이션과 관련된 정보 등을 얻는데 사용
- 컨텍스트 : 함수가 어떤 위치에서 사용되는지와 관련된 정보 제공

### 확장함수를 이용한 형태의 프로퍼티 위임

확장함수를 이용한 프로퍼티 위임을 통해 다음과 같은 형태로도 사용이 가능하다.

```Kotlin
val map: Map<String, String> = mapOf(  
    "name" to "kim",  
    "age" to "17"  
)

var name by map
println(name) // kim
```

위와 같은 형태로 사용이 가능한 이유는 코틀린 stdlib에 다음과 같이 확장함수로 getValue가 정의되어 있기 때문이다

```Kotlin
inline operator fun <V, V1 : V> Map<in String, V>
	.getValue(thisRef: Any?, property: KProperty<*>): V1 =
	getOrImplicitDefault(property.name) as V1
```

### 코틀린 stdlib에 선언되어 있는 Property Deligater

- lazy
- Delegates.observable
- Delegates.vetoable
- Delegates.notNull
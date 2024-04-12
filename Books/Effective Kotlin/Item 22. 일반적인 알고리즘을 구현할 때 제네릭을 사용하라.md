### 제네릭 함수

아규먼트로 함수에 값을 전달할 수 있는 것처럼, 타입 아규먼트를 사용하면 함수에 타입을 전달할 수 있다.

```Kotlin
inline fun <T> Iterable<T>.filter(
	predicate: (T) -> Boolean
): List<T> {
	val destination = ArrayList<T>()
	for(element in this) { 
		if(predicate(element)) {
			destination.add(element)
		}
	}
	return destination
}
```

타입 파라미터 정보는 컴파일러가 타입과 관련된 정보를 조금이라도 더 정확하게 추측할 수 있도록 해준다. 

### 제네릭 제한

타입 파라미터의 중요한 기능 중 하나는 타입의 서브타입만 사용하게 제한하는 것이다. 다음 코드를 보자.

```Kotlin
fun <T : Comparable<T>> Iterable<T>.sorted(): List<T> {
	/*...*/
}

fun <T, C : MutableCollection<in T>>
Iterable<T>.toCollection(destination: C): C {
	/*...*/
}
```

타입에 제한이 걸리기 때문에 내부에서 해당 타입이 제공하는 메서드를 사용할 수 있다. 예를 들어 T를 `Iterable<Int>` 의 서브 타입으로 제한한다면, T타입을 기반으로 반복처리가 가능하고 반복 처리 때 사용되는 객체가 `Int` 라는 것을 알 수 있다.
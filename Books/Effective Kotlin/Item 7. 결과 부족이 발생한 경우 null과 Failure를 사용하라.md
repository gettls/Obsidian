##  원하는 결과를 받지 못하는 상황에 대한 처리

함수를 호출했을 때 예상하는 결과와 다른 결과를 리턴해야만 하는 상황에 놓일 수 있다. 예를 들어서 다음과 같다.

- 조건에 맞는 요소가 없는 경우
- 네트워크 문제로 서버가 원하는 데이터를 받아오지 못한 경우

보통 이런 경우 결과를 다루는 방법은 크게 두 가지로 나뉠 수 있다.

- null 또는 실패를 나타내는 sealed 클래스(보통 `Failure`라는 이름 사용) 리턴
- 예외 throw

여기서 기억해야 할 점은 예외는 정보를 전달하는 방법으로 사용해서는 안된다는 것이다. 예외는 특별한 상황을 나타내기 위한 용도로 사용되어야 한다. 그 이유는 다음과 같다.

- 많은 개발자는 예외가 전파되는 과정을 제대로 추적하지 못한다.
- 코틀린의 모든 예외는 `unchecked`예외이고, 사용자가 예외를 제대로 처리하지 않을 수 있다. 
- 예외는 예외적인 상황을 처리하기 위해 만들어졌기 때문에 명시적인 테스트만큼 빠르게 동작하지 않는다.
- `try-catch` 블록 내부에 코드를 배치하면 컴파일러가 할 수 있는 최적화가 제한된다.

#### `null` 과 `Failure`로 사용으로 얻을 수 있는 이점

`null`과 `Failure`를 사용하면 명시적이고 효율적이며 간단하게 처리할 수 있다. 따라서 충분히 예측할 수 있는 범위의 오류는 `null`과 `Failure`를 사용하고, 예측하기 어려운 범위의 오류는 예외를 throw해서 처리하는 것이 좋다. 

```Kotlin
sealed class Result<T>
class Success<out T>(val Result: T): Result<T>()
class Failure(val throwable: Throwable): Result<Nothing>()
class JsonParsingException: Exception()
--- 
// null 사용
inline fun <reified T> String.readObjectOrNull(): T? {
	//...
	if(incorrectSign) {
		return null
	}
	//...
	return result
}
---
// sealed class 사용
inline fun <reified T> String.readObject(): Result<T> {
	//...
	if(incorrectSign) {
		return Failure(JsonParingException())
	}
	//...
	return Success(result)
```

위와 같이 처리를 하게 되면 예외를 놓치기 어렵고 예외 상황을 간단하게 처리할 수 있다.

- null 사용
```Kotlin
val age = userText.readObjectOrNull<Person>()?.age ?: -1
```
- sealed class 사용
```Kotlin
val person = userText.readObject<Person>()
val age = when(person) {
	is Success -> person.age
	is Failure -> -1
}
```

#### null 과 sealed class 의 차이점 ?

- sealed class : 추가적인 정보를 제공해야 하는 경우 사용
- null : 그렇지 않은 대부분의 경우, Failure를 처리할 때 정보를 사용할 수 있음

함수는 예상할 수 있는 경우와 예상할 수 없는 경우로 구분될 수 있는데, `List`를 사용하게 되면 이 두가지 경우 모두를 포함할 수 있다. 이런 경우 `List`는 보통 `get` 과 `getOrNull`을 사용한다.

- `get`: 만약 요소를 추출할 수 없다면 IndexOutOfBoundsException를 발생시킨다.
- `getOrNull`: out of range 오류가 발생할 수 있을 때 사용하며, 이 경우 null을 리턴하게 된다.

개발자는 항상 본인이 요소를 안전하게 추출할 것이라고 예측하기 때문에 nullable를 리턴하는 것은 위험하다. 즉, null이 발생할 수도 있다고 예측할 수 있도록 `getOrNull`등을 사용하는 것이 좋다.
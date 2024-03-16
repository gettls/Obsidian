
# 다양한 예외 활용 방법
- `require` : argument를 제한할 수 있다
- `check` : 상태와 관련된 동작을 제한할 수 있다
- `assert` : 어떤 것이 true인지 확인할 수 있다. (테스트 모드에서만 동작)
- `return`또는 `throw`와 함께 사용되는 `Elvis`연산자

### Argument 체크

일반적으로 Argument 값에 대한 제한을 위해서는 `require`블록을 많이 사용한다. `require`에 선언된 조건을 만족하지 못하면 `IllegalArgumentException`을 throw한다.

```Kotlin
fun factorial(n: Int): Long {
	require(n >= 0)
	// ...
}
```

예외 발생시 출력시킬 메세지도 지정할 수 있다.

```Kotlin
fun factorial(n: Int): Long {
	require(n >= 0) {
		"Number is lower than 0"
	}
	// ...
}
```

### 상태 체크

어떤 구체적인 조건을 만족할 때마 함수를 사용해야 할 때가 있다. 예를 들어 재고가 있어야만 주문을 할 수 있다는 조건같은 경우이다.

```Kotlin
fun order(book: Book) {
	check(hasStock) {
		"has no stock"
	}
	// ...
}
```

`require`과 비슷하지만 해당 조건을 만족시키지 못하는 경우 `IllegalStateException`을 throw하고 `require`처럼 메세지를 지정할 수도 있다. 일반적으로 `require`블록 뒤에 위치시킨다.

### Assert 계열 함수 사용

단위 테스트를 통해 어떤 메서드를 테스트하는 상황이라고 가정하자. 모든 상황에 대해서 테스트를 진행하지만, 메서드 내부에서 해당 메서드가 제대로 동작하는 지를 테스트하고 싶은 경우도 존재한다.

```Kotlin
fun pop(num: Int = 1): List<T> {
	// ...
	assert(ret.size == num)
	return ret
}
```

이는 ***테스트모드에서만 동작***하기 때문에 단위테스트 대신 함수에서 사용할 수도 있다. (어플리케이션 실행 시에는 예외를 던지지 않는다.)

특정 상황이 아닌 모든 상황에 대해 테스트할 수 있고, 실행 시점에 정확하게 어떻게 되는지 확인할 수 있다. 그리고, 더 빠른 시점에 실패하게 만들 수 있어서 개발자가 오류를 추적하는데 도움을 준다.

### nullability와 smart casting

위에서 언급한 조건을 통과한다면 해당 조건은 그 이후로도 true 일 것이라고 가정하기 때문에 스마트 캐스팅이 이루어진다. 예를 들어 다음과 같은 식이다.

```Kotlin 
fun changeDress(person: Person) {
	require(person.outfit is Dress)
	val dress: Dress = person.outfit // person.outfit -> Dress 로 캐스팅
	// ...
}
```

null 을 체크할 때 굉장히 유용하다.

```Kotlin
class Person(val email: String?)

fun sendEmail(person: Person, message: String) {
	require(person.email != null)
	val email: String = person.email
	//...
}
```

null 을 체크할 때는 `requireNotNull`, `checkNotNull`을 활용해서 변수를 unpack 하는 것도 괜찮다.

```Kotlin
class Person(val email: String?)

fun sendEmail(person: Person, message: String) {
	val email: String = requireNotNull(person.email)
	//...
}

fun sendEmail(person: Person, message: String) {
	val email: String = checkNotNull(person.email)
	//...
}
```

Elvis 연산자를 활용한 방법도 존재한다.

```Kotlin
fun sendEmail(person: Person, message: String) {
	val email: String = person.email ?: return
	//...
}
```

Elvis 연산자의 경우 `run`을 활용해 로그를 남기는 방법을 선택할 수도 있다.

```Kotlin
fun sendEmail(person: Person, message: String) {
	val email: String = person.email ?: run {
		log("Email not sent, no email address")
		return
	}
	//...
}
```
### 이름 있는 아규먼트

이름 있는 아규먼트(name argument)는 보통 의미가 명확하지 않은 경우에 사용하게 된다. 다음 코드를 보자.

```Kotlin
val text = (1..10).joinToString("|")
```

`joinToString` 함수에 대해 알고 있었다면 `"|"`가 구분자임을 알 수 있지만, 모르는 사실이었다면 `"|"` 가 어떤 역할을 하는 파라미터인지 알 수 없을 것이다. 이런 경우, 이름 있는 아규먼트(named argument)를 사용해서 파라미터에 대한 정보를 명시해줄 수 있다.

```Kotlin
val text = (1..10).joinToString(seperator = "|")
```

혹은 다음과 같이 파라미터에 대한 정보를 제공하는 다른 방법도 존재한다.

```Kotlin
val seperator = "|"
val text = (1..10).joinToString(seperator)
```

#### 이름 있는 아규먼트의 장점

- 이름을 기반으로 값이 무엇을 의미하는지 알 수 있다
- 파라미터 입력 순서와 상관없다.

파라미터에 대한 정보가 불명확하면 파라미터에 대한 중요한 정보를 제공하지 못할 수 있다.

```Kotlin
sleep(100) // 100이 얼마나 sleep한다는 것인지 알 수 없다
sleep(timeMillis = 100) // 1. named argument를 통해 정보를 제공하거나,
sleep(Millis(100)) // 2. 객체를 이용하거나,
sleep(100.ms) // 3. 확장함수를 이용하는 방법도 있다.
```

#### 언제 사용하는 것이 좋을까 ?

다음과 같은 경우에 특히 더 추천한다.

1. 디폴트 아규먼트의 경우
2. 같은 타입의 파라미터가 많은 경우
3. 함수 타입의 파라미터가 있는 경우

### 1. 디폴트 아규먼트의 경우

프로퍼티가 디폴트 아규먼트를 가질 경우, 항상 이름을 붙여서 사용하는 것이 좋다. 일반적으로 함수의 이름은 필수 파라미터와 관련되어 있고, 디폴트 값을 갖는 옵션 파라미터의 설명이 명확하지 않기 때문이다.

### 2. 같은 타입의 파라미터가 많은 경우

파라미터의 타입이 같은 파라미터가 많다면, 실수로 잘못된 위치에 잘못된 파라미터를 넣었을 때 오류가 발생할 수 있다. 

## 3. 함수 타입 파라미터

함수 타입 파라미터는 보통 마지막 위치에 배치하는 것이 좋다. 함수 이름으로 함수 타입 아규먼트를 설명해주기도 한다. 예를 들어 `repeat`의 경우 뒤에 오는 람다가 반복될 블록임을 나타내기도 한다. 

```Kotlin
val view = linearLayout {
	text("Click below")
	button({/* 1 */},{/* 2 */})
}
```

위의 코드에서 `/* 1 */` 과  `/* 2 */` 중 어떤 부분이 빌더 부분이고, 어떤 부분이 클릭 리스너를 의미할까? 이런 경우 위치를 수정하면 훨씬 명확해질 수 있다.

```Kotlin
val view = linearLayout {
	text("Click below")
	button(onClick = {/* 1 */}){
		/* 2 */
	}
}
```

**여러 함수 타입의 옵션 파라미터가 있는 경우**

```Kotlin
fun call( before: ()->Unit = {}, after: ()->Unit = {}) {
	before()
	print("Middle")
	call()
}

call( { print("CALL") }) // CALLMiddle
call( { print("CALL") }) // MiddleCALL
```

이러한 경우 이름을 붙여서 사용하면 더 쉽게 이해할 수 있다.

```Kotlin
call( before = { print("CALL") }) // CALLMiddle
call( after = { print("CALL") }) // MiddleCALL
```

Reactive 라이브러리에서 굉장히 자주 사용하는 형태이다. 예를 들어 RxJava에서 Observable을 구독할 때 다음과 같은 함수를 설정한다.

- onNext : 각각의 아이템을 받을 때
- onError : 오류가 발생했을 때
- onComplete : 전체가 완료됐을 때

```Java
// Java
observable.getUsers()
		.subscribe((List<User> users) -> { // onNext
			// ...
		}, (Throwable throwable) -> { // onError
			// ...
		}, () -> { // onComplete
			// ...
		});
```

위와 같은 Java 코드는 Kotlin 에서는 이름 있는 아규먼트를 사용해 더 쉽게 표현할 수 있다.

```Kotlin
observable.getUsers()
	.subscribeBy(
		onNext = { users: List<User> -> {
			// ...
		},
		onError = { throwable: Throwable -> 
			// ...
		},
		onCompleted = {
			// ...
		})
```

참고로, Java의 subscribe 메서드를 Kotlin에서는 subscribeBy로 변경했는데 이는 RxJava가 Java로 작성되어 있고, Java 함수를 호출할 때는 이름 있는 파라미터를 사용할 수 없기 때문이다. 이름 있는 파라미터를 사용하기 위해서는 이처럼 함수를 활용하는 별도의 Kotlin 함수를 만들어 사용해야 한다.

```Kotlin
fun Int.factorial(): Int = (1..this).product()
fun Iterable<Int>.product(): Int =   
    fold(1) { acc, i -> acc*i }
```

위와 같이 팩토리얼을 구하는 함수를 생각해보자. 연산자 오버로딩을 통해 다음과 같이 구현할 수 있다.

```Kotlin
operator fun Int.not() = factorial()

print(10 * !6) // 7200
```

당연히 관례적으로 사용하는 함수를 위와 같이 헷갈리게 사용해서는 안된다.


### 의미가 명확하지 않은 경우

관례를 충족하는지 아닌지 확실하지 않을 때도 있다. 예를 들어서, 함수를 3배 한다(`*` 연산자)는 것은 어떤 의미일까?

어떤 사람은 다음과 같이 이 함수를 3번 반복하는 새로운 함수를 만들어 낸다고 생각할 수도 있다.

```Kotlin
operator fun Int.times(operation: () -> Unit): ()->Unit =
	{ repeat(this) { operation() } }

val tripleHello = 3 * { print("Hello") }

tripleHello() // HelloHelloHello
```

하지만 어떤 사람은 이러한 코드가 함수를 세 번 호출한다는 것을 쉽게 이해할 수 있을 것이다.

```Kotlin
operator fun Int.times(operation: () -> Unit) { 
	repeat(this) { operation() } 
}

3 * { print("Hello") } // HelloHelloHello
```

이렇게 의미가 명확하지 않다면 infix를 활용해서 이항 연산자 형태처럼 사용할 수 있다.

```Kotlin
infix fun Int.timesRepeated(operation: () -> Unit) = { 
	repeat(this) { operation() }
}

val tripleHello = 3 timesRepeated { print("Hello") }
tripleHello() // HelloHelloHello
```

톱레벨 함수를 사용하는 것도 좋다.

```Kotlin
repeat(3) { print("Hello") }
```

### 규칙을 무시해도 되는 경우

연산자 오버로딩 규칙을 무시해도 되는 중요한 경우가 있는데, 그건 바로 도메인 특화 언어(DSL, Domain Specific Language)를 설계할 때다.

```Kotlin
body {
	div {
		+"Some text"
	}
}
```

위의 경우 문자열 앞에 `String.unaryPlus`가 사용됐는데, 이렇게 코드를 작성해도 되는 이유는 이 코드가 DSL이기 때문이다.
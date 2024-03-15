
상태를 정의할 때는 변수와 프로퍼티의 스코프를 최소화하는 것이 좋다. 

- 프로그램을 추적하고 관리하기 쉽다. 어떤 시점에 어떤 요소가 있는지 알기 쉽기 때문이다.
- 변수의 스코프가 넓어서 다른 개발자가 잘못 사용할 수 있는 실수가 줄어든다.

좁은 스코프를 갖게 하기 위해서는 최대한 지역 변수를 사용하고, 반복문 안에서만 사용되는 변수는 반복문 내에서 선언해야 한다. 그리고 선언과 함께 초기화하는 것이 좋다. 변수를 선언과 같이 초기화하는 방법으로는 `if`, `when`,`try-catch`, `Elvis` 표현식 등을 활용할 수 있다.


### 프로퍼티를 초기화하는 예시

```Kotlin 
// 나쁜예
val user: User
if (hasValue) {
	user = getValue()
} else {
	user = User()
}

// 조금 더 좋은 예
val user: User = if(hasValue) {
	getValue()
} else {
	User()
}
```

여러 프로퍼티를 초기화하는 경우 구조분해 선언(destructuring declaration)을 활용하는 것이 좋다

```Kotlin
// 나쁜예
fun updateWeather(degress: Int) {
	val description: String
	val color: Int
	if (degress < 5) {
		description = "cold"
		color = Color.BLUE
	} else if (degress < 23) {
		description = "mild"
		color = Color.YELLOW
	} else {
		description = "hot"
		color = Color.RED	
	}
}

// 조금 더 좋은 예
fun updateWeather(degress: Int) { 
	val (description, color) = when {
		degress < 5 -> "cold" to Color.BLUE
		degress < 23 -> "mild" to Color.YELLOW
		else -> "hot" to Color.RED	
	}
}
```


# 캡처링

변수나 프로퍼티의 스코프 범위가 넓으면 다음과 같은 캡처 문제가 발생할 수도 있다. 다음 예시를 보자. 에라토스네테스를 구현하는 코드이다.
`
```Kotlin
val primes: Sequence<Int> = sequence {
	var numbers = generateSequence(2) {it + 1}
	
	var prime: Int
	while(true){
		prime = numbers.first()
		yield(prime)
		numbers = numbers.drop(1)
			.filter {it % prime != 0}
	}
}
---
print(primes.take(10)).toList()
// [2,3,5,6,7,8,9,10,11,12] -> 예상치 못한 결과
```

위 코드가 예상치 못한 결과가 도출된 이유는 sequence를 활용하면서 필터링이 지연되기 때문이다prime 값이 2로 설정되어 있을 때 필터링된 4를 제외하면, drop만 동작하기 때문에 그냥 연속된 숫자가 나와버리게 된다.
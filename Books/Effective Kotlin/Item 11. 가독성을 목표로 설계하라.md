# 가독성의 필요성

###  1. 인식 부하 감소
```Kotlin
// 구현 A
if(person != null && person.isAdult) {
	view.showPerson(person)
} else {
	view.showError
}

// 구현 B
person?.takeIf { it.isAdult }
	?.let(view::showPerson)
	?: view.showError()
```

B가 더 짧단 이유로 가독성이 더 좋아보일 수 있으나 읽고 이해하기가 어렵다. 가독성이란 것은 코드를 읽고 얼마나 빠르게 이해할 수 있는지를 의미한다. 이해하기 쉬운 일반적인 관용구를 사용하고, 이러한 관용구를 조합해서 코드를 작성하는게 가독성이 더 좋다.

짧은 코드는 읽기 쉬울 수 있지만, 익숙한 코드는 더 빠르게 읽을 수 있다

### 2. 극단적이 되지 않기

`let`은 잘못알고 사용한다면 예상하지 못한 결과를 낳을 수 있지만, 절대로 쓰면 안된다로 이해하면 안된다. 

```Kotlin
class Person(val name: String)
var person: Person? = null

fun printName() {
	person?.let {
		print(it.name)
	}
}
```


가변 프로퍼티는 쓰레드와 관련된 문제를 일으킬 수 있기 때문에 스마트 캐스팅이 불가능하다. 이런 경우 위와 같이 `let`은 nullable 가변 프로퍼티를 안전하게 사용하기 위해서 유용하게 쓰일 수 있다. 
- 연산은 아규먼트 처리 후로 이동시킬 때
- 데코레이터를 사용해 객체를 래핑할 때

두 예시를 아래의 코드로 살펴보자.
```Kotlin
students
	.filter { it.result >= 50 }
	.joinToString(seperator = "\n") {
		"${it.name} ${it.surname}, ${it.result}"
	}
	.let(::print)

var obj = FileInputStream("/file.gz")
	.let(::BufferedInputStream)
	.let(::ZipInputStream)
	.let(::ObjectInputStream)
	.readObject() as SomeObject
```

### 3. 컨벤션

가독성이 높은 코드를 생성하기 위해 기억해야할 것 이 몇가지가 있다. 예시를 통해 알아보자.

```Kotlin
operator fun String.invoke(f: ()->String): String =
	this + f()
infix fun String.and(s: String) = this + s

val abc = "A" {"B"} and "C"
print(abc) // ABC
```

위의 코드는 안좋은 예시다. 위의 코드가 어긴 규칙들은 다음과 같다.

- 연산자는 의미에 맞게 사용해야 한다. `invoke`를 이러한 형태로 사용하면 안된다.
- '람다를 마지막 아규먼트로 사용한다' 라는 컨벤션을 여기에 적용하면 코드가 복잡해진다. `invoke` 연산자와 함께 이런 컨벤션을 적용하는 것은 신중해야 한다.
- `and`라는 함수 이름이 실제 함수 내부에서 이루어지는 처리와 맞지 않는다.
- 문자열을 결합하는 기능은 이미 내장되어 있다. 이미 있는 것은 다시 만들 필요가 없다.
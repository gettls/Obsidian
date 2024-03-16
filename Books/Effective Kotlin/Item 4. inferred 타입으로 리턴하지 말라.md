# type inference 사용 시 주의할 점

타입 추론을 사용할 때는 몇 가지 위험한 부분들이 있다. 이런 위험한 부분을 피하기 위해서는 할당 때 inferred type은 정확하게 오른쪽에 있는 피연산자에 맞게 설정된다는 점을 기억해야 한다.

```Kotlin
open class: Animal
class Zebra: Animal()

fun main() {
	var animal = Zebra()
	animal = Animal() // 오류 Type mismatch!!
}
```

위와 같은 경우는 타입을 명시적으로 지정해줌으로써 간단하게 해결할 수 있다.

``` Kotlin
fun main() {
	var animal: Animal = Zebra()
	animal = Animal() // OK
}
```

리턴 타입은 API를 잘 모르는 사람에게 전달해줄 수 있는 중요한 정보다. 외부로 inferred type을 반환하는 것은 다양한 문제를 발생시킬 수 있기 때문에 타입을 지정해서 반환하도록 하자.
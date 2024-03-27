### 여러 개의 리시버

스코프 내부에 둘 이상의 리시버가 있는 경우 명시적으로 리시버를 호출해주는 것이 좋다.

```Kotlin
class Node(val name: String) {
	fun makeChild(childName: String) = 
		create("$name.$childName")
			.apply { print("Created ${name}") }

	fun create(name: String): Node? = Node(name)
}

fun main() {
	val node = Node("parent")
	node.makeChild("child")
}
```

위의 코드의 결과를 예상한다면, 아마 `Created parent.child` 가 출력될 것이라고 생각하겠지만 실제로는 `Created parent`가 출력된다. 

그 이유는 `apply` 함수 내부의 `${name}`이 this 를 받아 타입이 `Node?` (create 메서드의 result를 수신하기 때문)라서 이를 직접 사용할 수 없고, 이를 사용하려면 다음과 같이 unpack하고 호출해야 한다. 

```Kotlin
class Node(val name: String) {
	fun makeChild(childName: String) = 
		create("$name.$childName")
			.apply { print("Created ${this?.name}") }

	fun create(name: String): Node? = Node(name)
}

fun main() {
	val node = Node("parent")
	node.makeChild("child")
}
```

사실 이런 경우 `apply` 보다는 `also` 를 사용하는 것이 더 나은 방법이다.

```Kotlin
class Node(val name: String) {  
    fun makeChild(childName: String) =  
        create("$name.$childName")  
            .also { print("Created ${it?.name}") }  
  
    fun create(name: String): Node? = Node(name)  
}
```

일반적으로 `let`, `also` 를 사용하는 것이 nullable 값을 처리할 때 훨씬 좋은 선택지이다. 참고로 리시버를 명시하지 않으면 가장 가까운 리시버를 의미하게 되기 때문에, 만약 외부에 있는 리시버를 사용하고자 한다면 레이블(`@`)을 사용하면 된다.

```Kotlin
class Node(val name: String) {  
    fun makeChild(childName: String) =  
        create("$name.$childName")  
            .also { print("Created ${it?.name} in" +  // create를 가리킴
                    " ${this@Node.name}") }  // Node를 가리킴
  
    fun create(name: String): Node? = Node(name)  
}
```

이렇게 한다면, 코드를 안전하게 사용할 수 있을뿐 아니라 가독성도 향상된다.

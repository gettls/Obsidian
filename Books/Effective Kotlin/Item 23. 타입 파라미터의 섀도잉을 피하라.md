```Kotlin
class Forest(val name: String){
	fun addTree(name: String) {
		// ...
	}
}
```

위와 같이 프로퍼티와 파라미터의 이름이 같으면 파라미터는 외부 스코프의 프로퍼티를 가리게 되는데, 이를 섀도잉이라고 한다.

```Kotlin
interface Tree  
class Birch: Tree  
class Spruce: Tree  
  
class Forest<T: Tree> {  
    fun <T: Tree> addTree(tree: T) {  
        // ...  
    }  
}
```

위와 같이 타입 파라미터 섀도잉이 발생한다면, Forest와 addTree의 타입 파라미터는 독립적으로 동작한다. 보통 이런걸 의도하는 경우는 많지 않을 것이다.

```Kotlin
interface Tree  
class Birch: Tree  
class Spruce: Tree  
  
class Forest<T: Tree> {  
    fun addTree(tree: T) {  
        // ...  
    }  
}

val forest = Forest<Birch>()  
forest.addTree(Birch())  
forest.addTree(Spruce()) // ERROR, Type Mismatch
```

위와 같이 클래스 타입 파라미터인 T를 사용하게 변경하는 것이 좋다. 섀도잉을 주의하자.
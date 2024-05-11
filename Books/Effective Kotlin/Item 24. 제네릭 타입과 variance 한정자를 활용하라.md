```Kotlin
class Cup<T>
```

위와 같은 코드에서 T는 variance 한정자(`out`, `in`)이 없기 때문에 불공변성(invariant)이다. 이는 `Cup<Int>`와 `Cup<Number>`가 어떤 관계도 없다는 뜻을 의미한다.

만약에 어떠한 관련성을 원한다면 `out` 또는 `in` 이라는 variance 한정자를 사용할 수 있다.

- `out` : A가 B의 서브타입일 때 `Cup<A>`는 `Cup<B>`의 서브타입이라는 의미 (covariant, 공변성)
	```Kotlin
		class Cup<out T>  
		open class Dog  
		class Puppy: Dog()  
		
		fun main(){  
		    val a: Cup<Dog> = Cup<Puppy>() // OK 
		    val b: Cup<Puppy> = Cup<Dog>() // 오류
		}
	```
- `in` : A가 B의 서브타입일 때 `Cup<A>`가 `Cup<B>`의 슈퍼타입이라는 의미 (contravariant, 반변성)
	```Kotlin
		class Cup<in T>  
		open class Dog  
		class Puppy: Dog()  
		
		fun main(){  
		    val a: Cup<Dog> = Cup<Puppy>() // 오류
		    val b: Cup<Puppy> = Cup<Dog>() // OK
		}
	```

## 함수 타입

함수 타입의 모든 파라미터 타입은 contravariant이다. 그리고 모든 리턴 타입은 covariant이다.
```Koltin
(T1, T2) -> E
```
즉, 위와 같은 코드가 있을 때 `T1, T2`는 covariant으로 적용되고, `E`는 covariant로 적용된다.

## variance의 안정성

자바의 배열은 covariant이다. 자바의 배열이 covariant 속성을 갖고 있기 때문에 다음과 같은 문제가 발생한다.

```Java
Integer[] numbers = {1, 4, 2, 1};
Object[] objects = numbers;
objects[2] = "B"; // Runtime Error: ArrayStoreException
```

covariant 속성 때문에 `Object[]` 에 `Integer[]` 를 참조시킬 수 있어서 컴파일 시점에는 문제가 없었지만, 런타임 시점에 문제가 발생한다.

코틀린은 위와 같은 문제를 해결하기 위해 `Array` 를 invariant로 만들었다. 즉, `Array<Int>`를 `Array<Any>`로 바꿀 수 없다는 뜻이다.


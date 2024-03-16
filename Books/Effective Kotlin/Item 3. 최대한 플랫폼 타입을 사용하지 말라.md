
> **플랫폼 타입 ?** = 다른 언어에서 넘어와 Kotlin이 null 관련 정보를 알 수 없는 타입
> 
>Kotlin은 null-safety하다. 하지만 그렇지 않은 Java, C, C++ 같은 null-safety가 없는 언어와 연결해서 사용할 때 문제가 발생할 수 있다.
>예를 들어서 Java에서 String을 반환한다고 했을 때, `@Nullable`어노테이션이 붙어있다면 Kotlin은 이를 `String?`으로 인식할 것이고, `@NotNull`이 붙어있다면 `String`으로 취급할 것이다.
>이렇게 명시적으로 null 여부를 알려주는 어노테이션이 없다면 Kotlin은 nullable 여부를 확인할 수 없고, 이런 특수한 유형을 `Platform type`이라고 한다.


# Platform Type

코틀린은 null-safety 매커니즘으로 인해 NPE가 거의 발생하지 않는다. 하지만 null-safety 하지 않은 언어와 같이 사용할 때 문제가 발생할 수 있다. 

nullable 과 관련해서 문제가 가장 많이 발생하는 경우는 Java의 Generic타입이다. nullable 관련 정보를 나타내는 어노테이션(`@Nullable`, `@NotNull`)이 없다면 우리는 모든 타입을 nullable로 다루게 될 것이다. 따라서 리스트와 리스트 내부의 객체들이 null아 아니라는 것을 체크해야 한다.

```Kotlin
// Java
public class UserRepo {
	public List<User> getUsers() {
		// ***
	}
}

// Kotlin
val users: List<User> = UserRepo().users!!.filterNotNull()
```

`List<List<User>>` 를 리턴한다면 과정은 더 복잡해진다.

```Kotlin
val users: List<List<User>> = UserRepo().groupedUsers!!
		.map { it!!.filterNotNull() }
```

따라서 Java 코드를 직접 조작할 수 있다면 가능한 `@Nullable`과 `@NotNull`어노테이션을 붙여서 사용해야 한다.

```Java
import org.jetbrains.annotations.NotNull;

public class UserRepo {
	public @NotNull User getUser() {
		//...
	}
}
```


#### Platform Type의 문제점

플랫폼 타입의 문제점은 NPE가 발생하는 위치를 집어내기 힘들 수 있다는 점이다. 다음 코드를 보자.

```Kotlin
// Java
public class JavaClass {
	public String getValue() {
		return null;
	}
}

// Kotlin
fun statedType() {
	val value: String = JavaClass().value // NPE 발생!!
	// ...
	println(value.length)
}
fun platformType() {
	val value = JavaClass().value
	// ...
	println(value.length) // NPE 발생!!
}
```

보통 객체를 사용한다고 해서 NPE가 발생할 것이라고 생각하지 않기 때문에 오류를 찾아내는 시간이 오래 걸릴 수 있다. 

플랫폼 타입은 더 많은 위험을 발생시킬 수 있다. 인터페이스에서 다음과 같이 플랫폼 타입을 사용했다고 해보자.

```Kotlin
interface UserRepo {
	fun getUserName() = JavaClass().value
}
```

위와 같은 경우 inferred 타입(추론된 타입)이 플랫폼 타입이다. 즉, 누구나 nullable 여부를 지정할 수 있다는 것이다. 예를 들어서 어떤 사람이 nullable을 리턴하게 했는데 사용하는 사람이 nullable이 아닐 것이라고 받아들였다면 문제가 된다.

```Kotlin
class RepoImpl: UserRepo {
	override fun getUserName(): String? {
		return null
	}
}

fun main() {
	val repo: UserRepo = RepoImpl()
	val text: String = repo.getUserName() // NPE !!!
	print("User name length is ${text.length}")
}
```

플랫폼 타입은 위험을 전파시킬 수 있다는 문제가 있다. 따라서 해당 코드는 제거하고, 연결되어 있는 Java 코드에 nullable 여부를 지정하는 어노테이션을 활용하도록 해야한다.
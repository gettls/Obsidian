### Kotlin의 프로퍼티

Kotlin에서의 프로퍼티는 Java의 필드와 매우 비슷해보이지만, 서로 완전히 다른 개념이다.

```Kotlin
var name: String? = null
String name = null;
```

데이터를 저장한다는 점에서 같지만 프로퍼티에는 더 많은 기능이 있다.

```Kotlin
var name: String? = null
	get() = field?.toUpperCase()
	set(value) {
		if(!value.isNullOrBlank()){
			field = value
		}
	} 
```

위에서 `field`라는 식별자를 확인할 수 있는데, 이는 프로퍼티의 데이터를 저장해 두는 백킹 필드(backing field)에 대한 레퍼런스이다. `field` 식별자는 setter, getter의 디폴트 구현에 사용되기 때문에 따로 만들지 않아도 디폴트로 생성이 된다. (참고로 val 을 사용해 읽기 전용 프로퍼티를 만들 때는 `field`는 생성되지 않는다.)

var를 사용해서 만들어진 프로퍼티는 getter, setter를 정의할 수 있고, 이런 프로퍼티는 파생 프로퍼티(derived property)라고 부르며 자주 사용된다. 이처럼 Kotlin의 모든 프로퍼티는 디폴트로 캡슐화가 되어 있다.

```Kotlin
interface Person {
	val name: String
}
```

프로퍼티는 필드가 필요하지 않다. 오히려 프로퍼티는 접근자를 나타낸다. (val -> getter, var -> getter/setter) 따라서 코틀린은 위와 같이 인터페이스에도 프로퍼티를 정의할 수 있게 된다.

```Kotlin
open class SuperComputer {  
    open val theAnswer: Long = 42  
}  
  
class AppleComputer: SuperComputer() {  
    override val theAnswer: Long = 1_800_275  
}
```

SuperComputer.theAnswer를 AppleComputer가 오버라이드를 할 수 있는 이유도 val 은 getter라는 접근자를 나타내기 때문이다. 마찬가지로 같은 이유로 프로퍼티 위임도 가능하다

```Kotlin
val db: Database by lazy { connectToDb() }
```

코드에서 확인할 수 있는 점은, 프로퍼티는 필드가 아니라 접근자를 나타낸다는 점이다. 아래의 코드와 같이 함수를 대체해서 프로퍼티를 사용할 수도 있지만 상황에 맞게 사용여부를 결정해야만 한다.

```Kotlin
val Context.preference: SharedPreferences
	get() = PreferenceManager
		.getDefaultSahredPreferences(this)
```

### 함수를 사용해야 하는 경우

구체적으로 프로퍼티 대신 함수를 사용하는 것이 좋은 경우를 정리해보면 다음과 같다.

- 연산 비용이 높거나 복잡도가 O(1) 보다 큰 경우
	- 관습적으로 프로퍼티를 사용할 때 연산 비용이 많이 들어간다고 생각하지 않는다.
	- 연산 비용이 많이 들어간다면 예측할 수 있도록 함수를 사용하는 것이 좋다. 그래야 사용자가 이를 기반으로 캐싱 등을 고려할 수 있기 때문이다.
- 비즈니스 로직을 포함하는 경우
	- 관습적으로 프로퍼티가 로깅/리스너 통지 등의 단순한 동작 이상을 할 것이라고 생각하지 않는다.
- 결정적이지 않은 경우
	- 같은 동작을 연속적으로 두 번 했을 때 다른 값이 나올 수 있다면 함수를 사용하자
- 변환의 경우
	- `Int.toDouble()`과 같이 변환 함수를 프로퍼티로 만들면 오해를 불러일으킬 수 있다.
- getter에서 프로퍼티의 상태 변경이 일어나야 하는 경우
	- 관습적으로 getter에서는 프로퍼티의 상태 변화가 일어난다고 생각하지 않는다. 

반대로 상태를 추출/설정할 때는 프로퍼티를 사용해야 한다. 특별한 이유가 없다면 함수를 사용하면 안된다.

```Kotlin
// Do not use like this

class UserIncorrect {
	private var name: String = ""
	fun getName() = name
	fun setName(name: String) {
		this.name = name
	}
}

class UserCorrect {
	var name: String = ""
}
```

프로퍼티는 상태 집합을 나타내고, 함수는 행동을 나타낸다고 생각한다.
# null 처리하는 3가지 방법

null은 값이 부족하다는 것을 나타낸다. 즉, 프로퍼티가 null이라는 뜻은 값이 설정되지 않았거나, 제거되었다는 것을 나타낸다. 기본적으로 nullable은 다음과 같은 3가지 방법으로 처리하게 된다.

-  `?.`, 스마트캐스팅, Elvis 연산자 등을 활용해 안전하게 처리하기
- 예외 throw
- 함수나 프로퍼티를 리팩토링해서 null이 나오지 않게 변경한다.

## null 안전하게 처리하기

null 을 안전하게 처리하는 방법으로는 크게 `안전 호출(safe call)`과 `스마트 캐스팅`이 있다.

```Kotlin
printer?.print() // 안전 호출
if (printer != null) printer.print() // 스마트 캐스팅
```

위 두 방법 모두 null이 아닐 때 print() 함수를 호출한다. 이 외에 Elvis 연산자를 사용한 방법도 많이 사용한다.

```Kotlin
val printerName1 = printer?.name ?: "Unnnamed"
val printerName2 = printer?.name ?: return
val printerName3 = printer?.name ?: 
	throw Error("Printer must be named")
```

컬렉션도 null을 처리하기 위한 메서드를 제공한다. 예를 들어 `Collection<T>.orEmpty()` 확장 함수를 사용하면 nullable이 아니라, `List<T>` 를 리턴받는다. 컬렉션은 값이 부족할 때 null 이 아닌 빈 컬렉션을 사용하기 때문이다.

스마트 캐스팅은 코틀린의 규약 기능(constracts feature)를 지원한다. 

```Kotlin
val name = readLine()
if (!name.isNullOrBlank()) {
	println("Hello ${name.toUpperCase()}") // String으로 스마트캐스팅
}

val news: List<News>? = getNews()
if (!news.isNullOrEmpty()) {
	news.forEach { notifyUser(it) } // List로 스마트캐스팅
}
```

> **방어적 프로그래밍**과 **공격적 프로그래밍**
> - 방어적 프로그래밍 : 모든 가능성을 올바르게 처리하는 방법으로, 예를 들면 'null 일 때 출력하지 않기' 등이 있다. 상황을 처리할 수 있는 올바른 방법이 존재할 때 좋다.
> - 공격적 프로그래밍 : 모든 가능성을 모두 예상할 수는 없기 때문에, 예상치 못한 상황이 발생하면 개발자에게 이를 알려 수정하게끔 만드는 방법이다. 예를 들어 require, check, assert 등의 도구가 존재한다.

## 오류 throw 하기

메서드나 프로퍼티를 사용하는 입장에서 null 이 발생할 수 있음을 생각하지 못했다면 위의 `printer?.print()`에서 메서드가 호출되지 않아 이상함을 느낄 것이다. 이런 경우 에러를 찾는 시간이 오래 걸릴 수 있기 때문에 `당연히 그럴 것이다`라고 생각되는 부분에 오류를 강제로 발생시켜 개발자가 이를 빠르게 인지하게 하는 방법이다. 보통 `throw`,`!!`,`requireNotNull`,`checkNotNull` 등을 활용한다.

## not-null assertion(!!)과 관련된 문제

nullable을 처리하는 방법 중 가장 간단한 방법은 어쩌면 not-null assertion(`!!`)일 것이다. 다만 이를 남용하면 Java에서의 문제와 같은 문제가 발생할 수 있다. null 이 아닐줄 알고 다루면 NPE가 발생하기 때문이다.

`!!` 은 타입은 nullable이지만, null이 나오지 않는다는 것이 거의 확실한 상황에서 많이 사용한다. 하지만, 현재 확실하다고 미래에 확실하다는 뜻은 아니다. 

`!!`는 NPE를 던지는데, `sealed Class`를 활용해 정보를 포함한 예외를 던지는 것이 더 나은 방법이다. 일반적으로는 `!!` 사용을 금하고, 다른 방법을 생각해봐야 한다.

## 의미 없는 nullability 피하기

nullability는 어떻게든 적절하게 처리되어야 하기 때문에 많은 리소스가 소요된다. 하지만, null은 중요한 정보를 제공하기 위해 사용될 수 있다. 그렇기 때문에 적절하게 nullability를 판단하는 것은 중요하다. 다만 불필요한 코드를 낭비하기 위해 nullability를 피할 수 있는 방법을 소개한다.

- 클래스에서 nullability에 따라 여러 함수를 만들어 제공할 수 있다. 예를 들어 `List<T>.get()`과 `List<T>.getOrNull()`이 있다.
- 어떤 값이 클래스 생성 이후 확실하게 설정된단 보장이 있다면 `lateinit` 프로퍼티와 `notNull` 델리게이트를 사용하라
- 빈 컬렉션 대신 null 을 리턴하지 마라. null과 빈 컬렉션이 의미하는 바는 완전히 다르다.
- `nullable enum` 과 `None enum`은 완전히 다른 의미이다. `null enum`은 별도로 처리해야 하지만, `None enum`은 정의에 없으므로 필요한 경우 사용하는 쪽에서 추가해서 활용할 수 있다는 의미를 갖는다.

---
## lateinit 프로퍼티와 notNull 델리게이트

```Kotlin
class UserControllerTest {
	private var dao: UserDao? = null
	//...
	@BeforeEach
	fun inti() {
		dao = mockk()
		//...
	}
	//...
}
```

위의 코드처럼 nullable인 것을 null로 바꾸는 작업은 불필요한 코드를 생산한다. 테스트 전에 초기화 될 것이 명확하기 때문이다.

```Kotlin
class UserControllerTest {
	private lateinit var dao: UserDao 
	//...
	@BeforeEach
	fun inti() {
		dao = mockk()
		//...
	}
	//...
}
```

`lateinit` 한정자를 사용한다면 이런 불필요한 코드를 방지할 수 있다. 물론, `lateinit`한정자에도 비용이 발생한다. 예를 들어 초기화 이전에 값을 사용하면 예외가 발생한다. 하지만 이는 알아야될 사실을 개발자가 인지한 것이기 때문에 오히려 좋은 일이 된다.

`lateinit`과 `nullable`에는 다음과 같은 차이가 있다.

- `!!` 연산자로 언팩하지 않아도 된다.
- 어떤 의미를 나타내기 위해 null 을 사용하고 싶을 때 nullable로 바꿀 수 있다.
- 프로퍼티가 초기화된 이후에는 초기화되지 않은 상태로는 돌아갈 수 없다.

### `lateinit` 한정자를 사용할 수 없는 경우

`Int`,`Long`,`Double`,`Boolean`과 같은 기본 타입에 연결된 타입들은 `lateinit`한정자를 사용할 수 없다. 이런 경우엔 `lateinit`보다는 느리지만, `Delegate.notNull`을 사용한다.

```Kotlin
class DoctorActivity: Activity() {
	private var doctorId: Int by Delegates.notNull()
	private var fromNotification: Boolean by Delegates.notNull()

	override fun onCreate(savedInstancesState: Bundle?) {
		super.onCreate(savedInstanceState)
		doctorId = intent.extras.getInt(DOCTOR_ID_ARG)
		fromNotification = intent.extras
			.getBoolean(FROM_NOTIFICATION_ARG)
	}
}
```

혹은 지연초기화하는 형태로 다음과 같이 프로퍼티 위임(property delegation)을 사용할 수도 있다.

```Kotlin
class DoctorActivity: Activity() {
	private var doctorId: Int by arg(DOCTOR_ID_ARG)
	private var fromNotifictation: Boolean by
		arg(FROM_NOTIFICATION_ARG)
}
```

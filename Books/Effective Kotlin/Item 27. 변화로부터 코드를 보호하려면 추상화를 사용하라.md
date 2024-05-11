#### 들어가며

함수와 클래스 등의 추상화로 실질적인 코드를 추상화한다면, 사용자의 입장에서는 해당 코드가 어떻게 동작하는 지 몰라도 함수를 사용할 수 있다. 또한, 이를 사용하는 코드에 영향을 주지 않으면서 메서드를 유연하게 수정할 수도 있다.

가장 간단한 예제로 시작해 단계를 높여가며 추상화를 진행해보며 그 장점을 파악해보자.

### 1. 상수

리터럴 자체는 어떤 것도 설명하지 않는다. 따라서 코드에서 반복적으로 등장하는 리터럴은 문제가 될 수 있고, 이를 상수 프로퍼티로 추상화한다면 의미를 부여할 수도 있고, 값이 변경되어야 할 때 훨씬 쉽게 변경할 수 있다.

```kotlin
fun isPasswordValid(text: String): Boolean {
	if(text.length < 7) return false
	// ...
}
```

비밀번호의 최소 길이를 나타내는 `7`을 상수 프로퍼티로 추상화시켜보자.

```kotlin
val MIN_PASSWORD_LENGTH = 7

fun isPasswordValid(text: String): Boolean {
	if(text.length < MIN_PASSWORD_LENGTH) return false
	// ...
}
```

내부 로직을 모르는 개발자가 비밀번호의 최소 길이를 변경하려고 할 때도 쉽게 변경할 수 있게 된다.

### 2. 함수

어플리케이션을 개발하는데, 사용자에게 토스트 메시지를 자주 출력해야 하는 상황이라고 가정해보자.

```kotlin
Toast.makeText(this, message, duration).show()
```

위와 같은 코드를 반복적으로 호출하고 있다면, 아래와 같이 확장함수로 간단하게 추상화를 할 수 있다.

```kotlin
fun Context.toast(
	message: String,
	duration: Int = Toast.LENGHT_LONG
) {
	Toast.makeText(this, message, duration).show()
}

context.toast(message)

toast(message)
```

이렇게 되면, 토스트를 출력하는 방법이 변경되어도 확장함수만 수정하면 되기 때문에 유지보수성이 향상된다. 만약 토스트가 아니라 스낵바 형태로 출력 방식을 변경해야 한다면 어떻게 해야할까 ?

```kotlin
fun Context.snackbar(
	message: String,
	duration: Int = Snackbar.LENGHT_LONG
) {
	// ...
}
```

위와 같은 확장함수를 생성한 후에, `Context.toast` 를 모두 `Context.snackbar`로 교체하면 된다. 하지만 이런 방식은 기존에 이 함수를 사용하고 있는 코드들에게 영향이 갈 수 있기 때문이다.

메시지를 출력하는 방식인 `Toast`, `Snackbar` 가 아니라, 메시지를 출력한다는 행위 자체에 의미를 두고 더 높은 레벨의 함수로 추상화를 한다면 이러한 문제를 해결할 수 있다.

```kotlin
fun Context.showMessage(
	message: String,
	duration: MessageLength = MessageLength.LONG
) {
	val toastDuration = when(duration) {
		SHORT -> Length.LENGTH_SHORT
		LONG -> Length.LENGHTH_LONG
	}
	Toast.makeText(this, message, toastDuration).show()
}

enum class MessageLength { SHORT, LONG }
```

이렇게 함수를 통해 추상화를 진행할 수 있지만, 함수는 상태를 유지하지 않기 때문에 함수를 통한 추상화는 제한이 많다. 또한 함수 시그니처를 변경하면 프로그램 전체에 큰 영향을 줄 수 있다. 구현을 추상화할 수 있는 방법으로는 더 강력한 클래스가 있다.

### 3. 클래스

위의 메시지 출력 예제를 클래스로 추상화해보자.

```kotlin
class MessageDisplay(val context: Context) { 
	fun show(
		message: String,
		duration: MessageLength = MessageLength.LONG
	) {
		val toastDuration = when(duration) {
			SHORT -> Length.SHORT
			LONG -> Lenght.LONG
		}
		Toast.makeText(context, message, toastDuration).show()
	}
}

enum class MessagLength { SHORT, LONG }

// use
val messageDisplay = MessageDisplay(context)
messageDisplay.show("Message")
```

클래스를 통한 추상화가 더 강력한 이유는 상태를 가질 수 있고, 더 많은 함수를 가질 수 있기 때문이다.                                     
### 4. 인터페이스

클래스를 인터페이스 뒤에 숨긴다면 더 많은 자유를 얻을 수 있다. 클래스의 가시성을 제한하고, 인터페이스를 통한 API를 제공함으로써, 별도의 걱정 없이 구현체를 변경할 수 있게 된다. 즉, coupling을 줄일 수 있게 되는 것이다.

```kotlin
interface MessageDisplay {
	fun show(
		message: String,
		duration: MessageLength = LONG
	)
}

class ToastDisplay(val context: Context): MessageDisplay {
	override fun show (
		message: String,
		duration: MessageLength
	) {
		val toastDuration = when(duration) {
			SHORT -> Length.SHORT
			LONG -> Length.LONG
		}
		Toast.makeText(context, message, toastDuration).show()
	}
}

enum class MessageLength { SHORT, LONG }
```

이렇게 구현체를 인터페이스 뒤에 숨긴다면, 태블릿에서 토스트를 출력시킬 수도 있고, 스마트폰에서 스낵바를 출력시킬 수도 있다. AOS, iOS, Web에서 공유하는 공통 모듈에서도 MessageDisplay를 사용할 수 있게 된다.

그리고 인터페이스를 Faking 하는 것이, 클래스는 Mocking 하는 것보다 간단하다는 것도 장점 중 하나다. 별도의 Mocking 라이브러리를 사용하지 않고도 테스트 코드를 자유롭게 작성할 수 있게 된다.

```kotlin
val messageDisplay: MessageDisplay = TextMessgaeDisplay()
```

마지막으로, 위와 같이 `선언`과 `사용`이 분리되어 있기 때문에 실제 구현체를 자유롭게 변경할 수 있다. 

## ID 만들기 (nextId)

추가적인 예를 하나 더 들어보자. 어떤 서버에서 고유 ID를 사용해야 하는 상황을 가정해보자. 가장 간단한 방법은 다음과 같을 것이다.

```kotlin
val nextId: Int = 0

// use
val newId = nextId++
```

상수를 사용한 방법인데 이러한 방법은 다음과 같은 문제가 있을 수 있다.

- 이 코드의 ID는 항상 0 부터 시작한다.
- thread-safe 하지 못하다.

그래도 이 방법을 사용해야 한다면 메서드로 추상화시켜 이후에 있을 변경으로부터 코드를 보호할 수 있다.

```kotlin
val nextId: Int = 0
fun getNextId(): Int = nextId++

// use
val newId = getNextId()
```

변경으로부터는 보호되지만, 타입 변경에는 대응할 수 없다. 미래에 ID를 문자열로 변경해야 한다는 요구사항이 있다면 이에 대응할 수 없게 된다. 이를 최대한 방지하기 위해서는 클래스를 사용하는 방법이 존재할 수 있다.

```kotlin
data class Id(private val id: Int)

private var nextId: Int = 0
fun getNextId(): Id = Id(nextId++)
```

더 많은 추상화는 더 많은 자유를 주지만, 이를 정의하고, 사용하고, 이해하기 어렵게 만들기도 한다.

## 추상화의 문제

추상화는 자유를 주지만 코드를 이해하고 수정하기 어렵게 만들기도 한다. 어떤 방식으로 추상화를 하려면 코드를 읽는 사람이 해당 개념을 배우고, 잘 이해해야 한다. 또 다른 방식으로 추상화를 하려면 또 해당 개념을 배우고, 잘 이해해야 한다. ==추상화도 비용이 발생한다는 의미이다.==

#### 어떻게 균형을 맞출 수 있을까?

추상화의 레벨 설정에 대한 해답은 다음과 같은 요소에 따라 달라질 수 있다.

- 팀의 크기
- 팀의 경험
- 프로젝트 크기
- feature set
- 도메인 지식

다양한 요소에 의해 추상화의 정도가 달라질 수 있고, 적절한 균형을 찾는 일은 엄청난 경험이 있어야만 할 수 있는 일이다. 그래도 사용할 수 있는 몇 가지 규칙을 정리해보자면 다음과 같다.

- 많은 개발자가 참여하는 프로젝트는 이후에 객체 생성/사용 방법을 변경하기 어렵다. 따라서 추상화 방법을 사용하는 것이 좋다.
- 의존성 주입 프레임워크를 사용하면, 생성이 얼마나 복잡한 지는 신경쓰지 않아도 괜찮다
- 테스트를 하거나, 다른 어플리케이션을 기반으로 새로운 어플리케이션을 만든다면 추상화를 사용하는 것이 좋다.
- 프로젝트가 작고 실험적이라면, 추상화를 하지 않고 직접 변경해도 괜찮다. 문제가 발생한다면 최대한 빨리 직접 변경하면 된다.

항상 무엇인가 변화할 수 있다고 생각해야 한다.


# Day 1

**최상위 프로퍼티**
```kotlin
val PORT_NUMBER = 1888

class Clazz {
  // ...
}
```
- `PORT_NUMBER` 를 사용하면 getter가 생긴다. (var는 getter, setter 둘 다 생김)
- 이런 상수 프로퍼티를 조금 더 자연스럽게 사용하기 위해서는 `public static final` 로 컴파일되도록 유도해야 한다. `const`를 사용하면 컴파일 시 `public static final int PORT_NUMBER = 1888` 로 컴파일된다. (`const val PORT_NUMBER`)

**확장 함수**
```kotlin
  [수신객체타입]                  [수신객체]  [수신객체]
fun String.lastChar() : Char = this.get(this.length-1)
```
- 확장 함수를 사용하면 멤버 메서드인 것처럼 사용할 수 있게 해준다. 
```kotlin
fun String.lastChar() : Char = get(length-1)
```
- 심지어 멤버 메서드에서 `this` 를 생략하는 것처럼 확장함수에서도 `this`를 생략할 수 있다.
- 확장 함수에서는 클래스 내부에서 선언된 `protected, private` 변수에는 접근할 수 없기 때문에 캡슐화를 깰 것이란 우려도 없다.
- 예를 들어 어떤 비즈니스 로직에서 중복되는 함수들을 함수로 추출해 중복을 줄이고 싶지만, 클래스의 멤버 메서드로 넣기에는 애매한 경우에 사용할 수 있다.

**확장 프로퍼티**
```kotlin
val String.lastChar: Char
	get() = get(length-1)
```
- 바로 위의 `확장함수`를 `확장프로퍼티`로 만들었다.  

**vararg**
```kotlin
fun listOf<T>(vararg values: T): List<T> {...}
```
- `vararg`는 가변 인자다. 자바에서 `...`으로 선언하던 것과 같다. 다만, 자바에서는 배열을 그대로 넘겨줘도 괜찮았지만, 코틀린에서는 원소들이 인자로 들어갈 수 있게 풀어서 전달해야 한다.
```kotlin
fun main(args: Array<String>) {
	val list = listOf(*args)
}
```
- 코틀린에서는 드 연산자 `*` 를 제공하고, 이 연산자는 배열의 내용을 풀어준다.

**문자열 파싱**
```kotlin
# JAVA
"a.b.c-d".split(".") -> []
# Kotlin
"a.b.c-d".split(".") -> [a, b, c-d]
```
- 코틀린에서는 `split` 에 들어가는 문자를 문자로 인식을 하지만, 자바에서는 정규표현식으로 인식힌다.
- 코틀린에서 정규표현식을 사용할 때는 `Regex`라는 별도의 객체를 사용하자

---

# Day 2

**인터페이스 구현**
```Kotlin
class Button: Clickable, Focusable {
	override fun Click() = println("...")
	override fun showOff() {
		super<Clickable>.showOff()
		super<Focusable>.showOff()
	}
}
```
- 만약 시그니처가 중복되는 경우, `super`와 `<>`를 사용해 타입을 명시적으로 지정한다.

**open, final, abstract 변경자**
- Kotlin은 기본적으로 모든 클래스가 `final` 이다. 명시적으로 `open` 변경자를 사용하지 않는다면 상속할 수 없다.
- Interface의 경우 모든 멤버 메서드가 열려있다.
- `override`로 상속한 메서드의 경우 기본적으로 열려있다.
- `abstract`의 경우 자바와 동일하게 인스턴스화할 수 없다. `abstract class`의 경우 멤버에 기본적으로 `open`이 붙는다.

**가시성 변경자**
- Kotlin에는 자바의 `package-private`가 없고 `internal`이라는 새로운 가시성 변경자가 생겼다.
	- `internal`: 같은 모듈에서만 보임. (모듈 = 같이 컴파일되는 단위를 의미)
```kotlin
interface Focusable {  
}  
  
internal open class Visibility : Focusable {  
    private fun yell() = println("yell")  
    protected fun whisper() = println("whisper")  
}  
  
fun Visibiliy.giveSpeech() = { // COMPILE ERROR - (1)
    yell() // COMPILE ERROR - (2)
    whisper() // COMPILE ERROR - (3)
}
```
(1): `internal class`를 `public` 멤버로 노출하려고 함
(2): `private` 멤버를 클래스 내에서 접근하지 않았음 (클래스 밖에서 접근)
(3): `protected` 멤버를 하위클래스가 아니라 클래스 외부에서 접근

**중첩 클래스**
```java
public interface State extends Serializable {}
public interface View {
	State getCurrentState();
	void restoreState(State state);
}
public class Button implements View {
	@Override
	public State getCurrentState() {...}
	@Override
	public void restoreState(State state) {...}
	public class ButtonState implements State {...} // 직렬화 X
}
```
- 자바에서는 `inner class`가 `outer class`의 묵시적 참조를 가진다. 따라서 `ButtonState`를 직렬화하려고 하면 묵시적 참조 때문에 예외가 발생한다.
- 따라서 이런 경우, `inner class`를 `static public class`로 변경해줘야 직렬화를 할 수 있다.
```Kotlin
interface State: Serializable
interface View {
	fun getCurrentState(): State
	fun restoreState(state: State) {}
}
class Button: View {
	override fun getCurrentState(): State {...}
	override fun restoreState(state: State) {...}
	class ButtonState: State {...}
}
```
- 같은 상황에서 Kotlin은 예외가 발생하지 않는다. 기본적으로 변경자가 없는 `inner class`는 `static`으로 선언되기 때문이다.
- 만약 외부 클래스에 대한 참조를 갖게 만들고 싶다면 `inner` 변경자를 사용해야 한다.
```Kotlin
class Outer {
	inner class Inner {
		fun getOuterReference(): Outer = this@Outer
	}
}
```
- 바깥쪽 클래스에 대한 참조를 제공하고 싶다면 `@` 를 사용한다.
 
---
# Day3

**인터페이스 프로퍼티 구현**
```Kotlin
interface User {
	val nickname: String
}

class EmailUser(email: String): User { // (1)
	override val nickname: String
		get() = email.substringBefore('@') // 커스텀게터
}

class FacebookUser(accountId: Long): User { // (2)
	override val nickname: String = this.getNickname(accountId)
}
```
(1): `nickname` 변수를 호출할 때마다 함수를 호출해서 새로 만들어서 값을 가져온다
(2):`nickname` 변수는 객체를 생성할 때 만들어져, `뒷받침하는 필드`에 저장되어 있다가 그 값을 가져온다.

**뒷받침 필드 접근하기**
```kotlin
class User(name: String) {  
    var address: String = "unspecified"  
        set(value: String) {  
            println("""  
                [change] $field -> $value  
            """.trimIndent())  
            field = value  
        }  
}
```
- `field`라는 특수한 식별자를 통해 뒷받침 하는 필드에 접근할 수 있다.

**접근자 가시성 변경하기**`
```kotlin
class LengthCounter {  
    var counter: Int = 0  
        private set  
    fun addWord(word: String) {  
        counter += word.length  
    }  
}

fun main() {  
    val lengthCounter = LengthCounter()  
    lengthCounter.addWord("word")  
    println(lengthCounter.counter) // getter(public) OK
    lengthCounter.counter = 0 // setter(private) X -> COMPILE ERROR
}
```
- 위와 같이 `setter`의 가시성만 private으로 선언할 수 있다.

**by 키워드**
```Kotlin
  interface Calculator {  
    fun add(a: Int, b: Int): Int  
    fun minus(a: Int, b: Int): Int  
    fun multiply(a: Int, b: Int): Int  
    fun divide(a: Int, b: Int): Int  
}  

// by 사용 
class ProfessionalCalculator(  
    val innerCalculator: Calculator  
) : Calculator by innerCalculator {  
}  
  
class ArmatureCalculator (  
    val innerCalculator: Calculator  
): Calculator {  
    override fun add(a: Int, b: Int): Int {  
        return innerCalculator.add(a, b)  
    }  
  
    override fun minus(a: Int, b: Int): Int {  
        return innerCalculator.minus(a, b)  
    }  
  
    override fun multiply(a: Int, b: Int): Int {  
        return innerCalculator.multiply(a, b)  
    }  
  
    override fun divide(a: Int, b: Int): Int {  
        return innerCalculator.divide(a, b)  
    }  
}
```
- 보일러플레이트 코드를 컴파일러가 대신 생성해주기 때문에, 위의 코드를 보면 알 수 있듯 `Delegate` 패턴을 사용할 때 유용하게 사용할 수 있다.

**object 키워드**
- `object` 객체 선언
```Kotlin
object Payroll {
	val allEmployees = arrayListOf<Person>()
	fun calculateSalary() {
		// ...
	}
}

Payroll.allEmployees.add(Person(...))
Payroll.calculateSalary()
```
- `object`는 싱글턴을 생성할 때 사용한다.
- 싱글턴 객체는 생성자 선언없이 바로 멤버에 접근해서 바로 사용한다.

- `companion object` 동반 객체
```kotlin
class A {
	companion object {
		fun bar() {
			// ...
		}
	}
}

fun A.Companion.foo() {...} // 동반객체 확장함수
```

- 무명 객체(`anonymous object`) 선언
```kotlin
winodw.addMouseListener (
	object : MouseAdapter() {
		override fun mouseClicked(e: MouseEvent) {
			// ...
		}
		...
	}
)
```

# Day 4

**Sequence**
- 자바의 Stream과 동일한 개념, -> `지연연산`이 이루어짐
- `asSequence()` API를 통해 Sequence로 변환할 수 있음
```Kotlin
listOf(1,2,3,4).asSequence()
	.filter { it % 2 == 0 } 
	.map { it * it }
	.toList()
```
- `즉시연산` 
	- (1,2,3,4) 가 filter 모두 수행 -> (1,3) 가 map 수행 -> toList로 모임
- `지연연산` : 
	- (1) 이 filter -> map -> toList 수행
	- (2) 이 filter -> map -> toList 수행
	- ...
- 위와 같은 특성을 가지기 때문에, filter를 위에 두면 최적화를 시킬 수 있음

**람다 넘기기**
```kotlin
fun print(id: Long, runnable: Runnable) { 
	return runnable.run()
}
```
- 메서드의 매개변수로 람다를 넘길 수 있다. 여기서 말하는 람다에는, `함수형 인터페이스`, `SAM(Single Abstract Method)`가 있다.
- 람다와 변수 객체 생성
```kotlin
fun printLambda(id: Long, runnable: Runnable) {  
    return runnable.run()  
}

fun print(id: String) {  // - (1)
    printLambda(10) {  
        println(100) // 주변의 변수 사용 X
    }  
}

fun print(id: String) {  // - (2)
    printLambda(10) {  
        println(id) // 주변의 변수 사용 O
    }  
}  
```
- (1)번
	- `Runnable` 객체를 생성해서 변수에 저장해두고 이를 전역적으로 사용한다. 즉, 객체가 단 하나밖에 만들어지지 않는다.
- (2)번
	- 함수를 호출할 때마다 새로운 `Runnable`객체를 생성해서 이를 사용한다. 이처럼, 주변의 변수를 포착해서 사용하는 경우 객체가 매번 생성되기 때문에 이를 조심해야 한다.

**수신객체 지정 람다**
- `with`
```kotlin
val result = StringBuilder()  
with(result) {  
    append("my")  
    append("name")  
    append("is")  
    append("smc")  
    toString()  
}  
println(result)
```
- `with` 메서드의 파라미터는 두 개다. 첫 번째 파라미터는 수신객체이고, 두 번째 파라미터는 람다로, 첫 번째 파라미터로 들어온 수신객체의 함수를 지정한다.

- `apply`
```kotlin
val result2 = StringBuilder()  
result2.apply {  
    append("my")  
    append("name")  
    append("is")  
    append("smc")  
}.toString()  
println(result2)
```
- `with` 과의 차이점은 `apply`는 확장함수로 존재한다는 점이다. 즉, 파라미터로 수신객체를 받지 않는다.

# Day 5

- `
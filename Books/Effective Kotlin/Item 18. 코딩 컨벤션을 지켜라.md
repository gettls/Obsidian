### 코딩 컨벤션을 지켜야하는 이유

1. 어떤 프로젝트를 정해도 쉽게 이해할 수 있다
2. 다른 외부 개발자도 프로젝트의 코드를 쉽게 이해할 수 있다
3. 다른 개발자도 코드의 작동 방식을 쉽게 추측할 수 있다
4. 코드를 병합하고, 한 프로젝트의 코드 일부를 다른 코드로 이동하는 것이 쉽다.

코딩 컨벤션을 지킬 때 사용할 수 있는 유용한 도구들도 있다.

- Intellij 포매터 : 공식 코딩 컨벤션 스타일에 맞게 코드를 변경해준다. `Settings > Editor > Code Style > Kotlin` 에서 오른쪽 위에 있는 `Set from ...` 링크를 누른 뒤 메뉴에서 `Predefined style/Kotlin style guide`를 선택
- ktlint : 많이 사용되는 린터

### 자주 위반되는 클래스와 함수 선언 방식

많은 파라미터를 갖고 있는 함수와 클래스는 다음과 같이 각각의 파라미터를 한 줄씩 작성해야 한다.

```Kotlin
public fun <T> Iterable<T>.joinToString(
	separator: CharSequence = ", ", 
	prefix: CharSequence = "", 
	postfix: CharSequence = "", 
	limit: Int = -1, 
	truncated: CharSequence = "...", 
	transform: ((T) -> CharSequence)? = null
): String {  
	// ...
}
```

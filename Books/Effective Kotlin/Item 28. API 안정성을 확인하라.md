#### 들어가며

만약 자동차의 운전 방법이 모두 다르다면, 운전하는 방법을 배우는 일을 굉장히 귀찮은 일이 될 것이다. 프로그래밍도 똑같다. 일시적으로 사용하는 인터페이스를 배우는 것은 귀찮은 일이기 때문에, 우리는 최대한 안정적이고 표준적인 API를 사용하는 것을 선호한다. 그 이유는 다음과 같다.

- API가 변경될 때마다 여러 코드를 수동으로 업데이트 해줘야만 한다. 많은 요소가 API에 의존하고 있었다면 이는 큰 문제가 된다. 라이브러리의 작은 변경은 이를 활용하는 다른 코드들의 많은 부분들을 변경하게 만들기 때문에, 라이브러리가 변경되어도 이전 버전을 유지하는 경우가 많다. 하지만 그럴수록 점점 업데이트가 어려워지고 버그와 취약성 등이 발생할 수 있다. 따라서 개발자가 라이브러리를 안정적인 버전으로 업데이트 하는 것을 두려워하는건 매우 좋지 않다.
- 사용자는 새로운 API를 배워야 한다. 무언가를 새로 배우는 것은 굉장히 많은 리소스를 필요로 한다. 따라서 처음부터 안정적이지 않은 모듈을 많이 공부하는 것 보다는 안정적인 모듈부터 공부하는 것이 좋다.

### API 변경 사항 트래킹

많은 버져닝 시스템(Versioning System)이 있지만 일반적으로는 Semantic Versioning을 사용한다. 

- Major : 호환되지 않는 수준의 API 변경
- Minor : 이전 변경과 호환되는 기능을 추가
- Patch : 간단한 버그 수정

Major 버전이 0 인 경우는 초기 개발 전용 버전을 의미한다. 따라서 언제든 변경될 수 있고, 안정적이지 않다는 것을 의미한다. 베타버전이 오래 유지되는 것을 싫어하는 사람도 있지만, 코틀린도 1.0 까지 도달하는데 5년이라는 시간이 걸렸다.

안정적인 API에 새로운 요소를 추가할 때, 아직 해당 요소가 안정적이지 않다면 먼저 다른 브랜치에 해당 요소를 두는 것이 좋다. 일부 사용자가 이를 사용하도록 허락하려면 `Experimental` 메타 어노테이션을 사용하는 것이 좋다. 아직 해당 요소가 안정적이지 않다는 것을 알려주는 것이다.

```kotlin
@Experimental(level = Experimental.Level.WARNING)
annotation classs ExperimentalNewApi

@ExperimentalNewApi
suspend fun getUsers(): List<User> {
	// ...
}
```

안정적인 API의 일부를 변경해야 한다면, 전환하는 데 시간을 두고 `Deprecated` 어노테이션을 활용해야 한다.

```kotlin
@Deprecated("User suspending getUsers instead")
fun getUsers(callback: (List<User>)->Unit) {
	// ...
}
```

또한 직접적인 대안이 있다면 IDE가 자동 전환을 할 수 있도록 `ReplaceWith`를 붙여주는 것도 좋다.

```kotlin
@Deprecated("User suspending getUsers instead")
@ReplaceWith("getUsers()")
fun getUsers(callback: (List<User>)->Unit) {
	// ...
}
```
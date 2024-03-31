###  Knowledge

knowledge란 프로젝트를 진행할 때 정의한 모든 것을 의미한다. 예를 들면, 알고리즘의 동작 방식, UI의 형태, 우리가 원하는 모든 결과 등이 이에 해당한다. 프로그램에서 가장 중요한 knowledge를 뽑으라면 다음과 같은 두 가지를 뽑을 수 있다.

- logic: 프로그램이 어떻게 동작하는지와 어떻게 보이는 지를 의미한다
- common algorithm: 원하는 동작을 하기 위한 알고리즘

두 가지의 차이점은 시간에 따른 변화이다. 비즈니스 로직은 시간에 따라 계속해서 변화하지만, 공통 알고리즘은 크게 변하지 않는다. 

### 코드를 반복하는 것의 위험성

프로그램은 시간에 따라 계속해서 변화한다. 그 이유는 디자인의 표준이 변해서일수도 있고, 플랫폼이나 라이브러리가 변화해서일수도 있다. 기술의 표준은 빠르게 변화하고 우리는 이에 대응해야할 필요가 있다.

변화를 해야하는 시점에 가장 큰 걸림돌은 반복되어서 사용된 Knowledge들이다. 여러 부분에서 반복되어 있는 코드들을 변경하는 것은 어려운 일이다. 단순히 모든 코드를 공통적으로 변경하면 되지 않나 라는 생각을 할 수도 있지만, 그 코드의 일부가 과거에 약간 다른 방식으로 수정되어 있을 수도 있고 변수는 다양하다.  다행히도 개발자는 이러한 코드의 반복을 `추상화`를 통해 줄여나갈 수 있다.

그렇다면 **'코드는 절대 반복되어서 사용되면 안되는건가?'** 라는 물음이 생길 수 있다. 그에 대한 대답부터 말하자면, knowledge의 반복처럼 보이지만 실질적으로 다른 knowledge를 나타내면 extract를 하면 안된다는 것이다.

두 개의 코드가 같은 knowledge에서 온 것인지를 판단할 수 있는 방법은 **'함께 변경될 가능성이 높은가? 따로 변경될 가능성이 높은가'** 에 대한 질문을 생각해보는 것이다. 코드를 extract하는 이유는 변경을 쉽게 만들기 위함이기 때문에 이는 근본적인 질문이 될 수 있다.  또 한가지 유용한 방법으로는 **단일 책임 원칙**을 이용하는 것이다.

### 단일 책임 원칙

단일 책임 원칙이란, 클래스를 변경하는 이유는 단 한가지여야 한다(A class should have only one reason to change)라는 의미이다.  로버트 C. 마틴은 `두 actor가 같은 클래스를 변경하는 일은 없어야한다.`로 이를 비유한다. (여기서 actor는 개발자를 의미한다.)

예시를 들어보자. 

```Kotlin
class Student {
	// ...
	fun isPassing(): Boolean =
		calculatePointsFromPassedCourses() > 15
		
	fun qualifiesForScholarship(): Boolean =
		calculatePointsFromPassedCourses() > 30

	private fun calculatePointsFromPassedCourses(): Int {
		// ...
	}
}
```

- isPassing : 인증 관련 부서에서 만든 프로퍼티로, 학생이 인증을 통과했는지 나타냄
- qualifiesForScholarship : 장학금 부서에서 만든 프로퍼티로, 학생이 장학금을 받을 수 있는 포인트를 갖고 있는지 나타냄

만약 위와 같은 상황에서 다른 개발자가 덜 중요한 과목의 장학금 포인트를 줄여야 한다는 새로운 요구사항을 위해 `qualifiesForScholarship`을 수정하기 위해서 `calculatePointsFromPassedCourses`를 수정한다면 어떨까?

처음 코드를 작성한 사람은 `calculatePointsFromPassedCourses` 가 공통 알고리즘이라고 생각하고 extract를 했겠지만, `isPassing`와 `qualifiesForScholarship`는 서로 다른 knowledge에서 출발한 로직이어서 서로 관련이 없는 프로퍼티가 결과적으로 변화에 취약한 코드가 되었다.

단일 책임 원칙을 준수해서 책임에 따라 `StudentIsPassingValidator`, `StudentQualifiesForScholarshipValidator` 클래스를 구분해서 동작을 수행하게 했다면 어땠을까 생각해볼 수 있다. 아니면, Kotlin의 확장함수를 사용했다면 어땠을까 ?

단일 책임 원칙은 다음과 같은 두 가지 사실을 알려준다.

1. 서로 다른 곳에서 사용하는 knowledge는 독립적으로 변경할 가능성이 많다. 따라서 비슷한 처리를 하더라고, 완전히 다른 knowledge로 판단하는 것이 좋다.
2. 다른 knowledge는 분리해 두는 것이 좋다. 그렇지 않으면, 재사용해서는 안 되는 부분을 재사용하려는 유혹이 발생할 수도 있다.

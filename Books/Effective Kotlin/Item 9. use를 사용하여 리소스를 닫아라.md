## 명시적으로 닫아줘야 하는 리소스

JVM/코틀린에서 사용하는 자바 표준 라이브러리에는 사용 후 명시적으로 `close`를 선언해서 닫아줘야 하는 리소스들이 존재한다. 

- `InputStream`, `OutputStream`
- `java.sql.Connection`
- `java.io.Reader(FilterReader, BufferedReader, CSSParser)`
- `java.new.Socket`, `java.util.Scanner`

보통 `AutoCloseable`을 상속받는 `Closeable`을 구현하는 형태로 처리하고 있지만, 이런 가비지 컬렉션은 굉장히 느리고 리소스를 유지하는 비용이 많이 들기 때문에 명시적으로 `close` 메서드를 호출하는게 좋다.

### use 메서드

보통 `close`를 명시적으로 선언해서 리소스를 닫으려면 다음과 같이 `try-catch`로 묶어서 선언을 하게 된다.

```Kotlin
fun countCharactersInFile(path: String): Int {
	val reader = BufferedReader(FileReader(path))
	try { 
		return reader.lineSequence().sumBy { it.length }
	} finally {
		reader.close()
	}
}
```

하지만 위와 같은 방법을 사용하면 `close()` 가 동작하는 시점에 에러가 발생할 수 있고, 이를 처리하기 위해서는 복잡한 코드를 작성해야만 한다. 이렇게 복잡한 코드를 작성하는 대신 표준 라이브러리의 `use` 메서드를 사용하면 간단하게 해결할 수 있다.

```Kotlin
fun countCharactersInFile(path: String): Int {
	val reader = BufferedReader(FileReader(path))
	reader.use { 
		return reader.lineSequence().sumBy { it.length }
	}
} 
```

혹은 람다 매개변수 리시버로 전달되는 형태로 작성할 수도 있다.

```Kotlin
fun countCharactersInFile(path: String): Int {
	BufferedReader(FileReader(path)).use { reader ->
		return reader.lineSequence().sumBy { it.length }
	}
} 
```

파일을 한 줄씩 읽어서 처리하기 위해 `useLines` 함수도 지원한다. 메모리에 한 줄씩만을 유지하기 때문에 대용량 파일도 용이하게 처리할 수 있다.

```Kotlin
fun countCharactersInFile(path: String): Int {
	File(path).useLines { lines ->
		lines.sumBy { it.length }
	}
} 
```



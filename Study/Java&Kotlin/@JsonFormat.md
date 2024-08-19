```Text
@JsonFormat is a Jackson annotation that allows us to configure how values of properties are serialized or deserialized. For example, we can specify how to format `Date` and `Calendar` values according to a `SimpleDateFormat` format.
```

# @JsonFormat
- `@JsonFormat`은 JSON Serialization, Deserialization이 될 때, 해당 변수를 어떤 포맷으로 처리할 지를 결정할 수 있도록 해준다.

#### 예시
```java
// default
public class DTO {
	private Date createdDate; 
}
>>> {"createdDate":17182728166}
// JsonFormat
public class DTO {
	@JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd@HH:mm:ss.SSSZ")
	private Date createdDate; 
}
>>> {"createdDate":"2016-12-18@07:53:34.740+0000"}
```


https://www.baeldung.com/jackson-jsonformat
https://umbum.dev/1055/
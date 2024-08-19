
```
Annotation that can be used to either suppress serialization of properties (during serialization), or ignore processing of JSON properties read (during deserialization).  
Example:  
  // to prevent specified fields from being serialized or deserialized  
  // (i.e. not include in JSON output; or being set even if they were included)  
  @JsonIgnoreProperties({ "internalId", "secretKey" })  
  // To ignore any unknown properties in JSON input without exception:  
  @JsonIgnoreProperties(ignoreUnknown=true)  
   
Annotation can be applied both to classes and to properties. If used for both, actual set will be union of all ignorals: that is, you can only add properties to ignore, not remove or override. So you can not remove properties to ignore using per-property annotation.
```
# @JsonIgnoreProperties(ignoreUnknown=true)
- JSON 을 deserialization 하는 시점에 모르는 property가 있어도 에러를 내뱉지 않고 무시한다

#### 예시
```Kotlin
data class DTO{
	val name: String
}
```

```Json
{
	"name":"홍길동",
	"age":13
}
```

위와 같은 상황에서도 에러를 뱉지 않고, name에 대한 값만 사용을 한다.
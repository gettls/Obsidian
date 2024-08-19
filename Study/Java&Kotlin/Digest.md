```Java
public Mono<String> generateToken(final String queue, final Long userId){  
    MessageDigest digest = null;  
    try {  
        digest = MessageDigest.getInstance("SHA-256");  
        var input = "user-queue-%s-%d".formatted(queue, userId);  
        byte[] encodedHash = digest.digest(input.getBytes(StandardCharsets.UTF_8));  
  
        StringBuilder hexString = new StringBuilder();  
        for (byte aByte: encodedHash) {  
            hexString.append(String.format("%02x", aByte)); // ?  
        }  
        return Mono.just(hexString.toString());  
    } catch (NoSuchAlgorithmException e) {  
        throw new RuntimeException(e);  
    }  
}
```

위의 함수를 이해할 수가 없어서, 그리고 과거에 토큰을 만들 때 위와 같은 함수를 만들어본 기억이 있는데,
copy & paste로 함수를 만들었기 때문에 이번 기회로 공부를 하려고 한다.

---


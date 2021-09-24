## Java Regex 문자열 Boolean 매칭시키기

    문자열을 매칭시켜 Boolean 값을 리턴받는 방법을 알아보자

```java
String pattern = "(?=.*[^a-z]).*";

boolean res = Pattern.matches(pattern, 매칭시킬 문자열);
```

    위처럼 매칭시키면 res에 결과가 들어간다.

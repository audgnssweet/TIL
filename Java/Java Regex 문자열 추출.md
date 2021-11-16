## Java 에서 Regex를 이용한 Split 말고, 문자열 추출에 대해 알아보자.

    1S2D+3T 에서
    1S, 2D+, 3T 셋으로 문자를 추출하고자 한다.

    코드는 아래와 같다.

```java

Pattern pattern = Pattern.compile("(\\d+\\D+)");
Matcher matcher = pattern.matcher(string);

while (matcher.find()) {
    System.out.println(matcher.group());
}

```

    Pattern에 정규식을 넣어서 캡쳐하고싶은 문자열 패턴을 지정해준다.
    while(matcher.find()) 가 마치 scanner.hasNext() 와 같은 역할을 하여
    매칭되는 것이 없을 때까지 계속 문자열을 추출해준다.

    이들은 모두 java.util 패키지에 속해있다.

## 예시 2

"//;\n1;2;3" 에서 ; 와 1;2;3을 따로 추출하려고 한다.

```java
        Matcher matcher = Pattern.compile("//(.)\n(.*)").matcher(expression);
        if (matcher.find()) {
            String customDelim = matcher.group(1);
            String exp = matcher.group(2);
        }
```

위처럼 Group 번호를 명시해줌으로써 -> 소괄호 안이 하나의 group 번호

## String vs. StringBuilder vs. StringBuffer

    Java에는 대표적인 문자열 삼총사가 있다.
    이들의 차이점을 알아보자.

#### 불변성 (immutable)

    불변: String
    가변: StringBuilder, StrinbBuffer

    Java의 대표적인 자료형 String은 불변한다는 특징이 있다.
    불변한다는 것은, String 자료형으로 처음에 한번 선언하고나면, 문자를 바꿀 수 없다는 의미와 같다.

    예를 들어 아래와 같은 코드가 있다고 치자.

```java
    public static void main(String[] args) {
        String example = "example";
        example.replace("ex", "my");
    }
```

    위와 같은 코드가 있다고 example이라는 string이 myample로 바뀌기를 기대한다.
    그러나 바뀌지 않는다. string 자체가 바뀌는 것이 아니라 새로운 메모리 공간을 할당하여 myample 문자열을 만들고
    반환한다.

    그렇다면 위는 어떻게 해결해야할까?

```java
    public static void main(String[] args) {
        String example = "example";
        example = example.replace("ex", "my");
    }
```

    위처럼 재할당을 해주면 된다. String을 가지고 변화를 요한다면, 항상 재할당이 필요하다.

    그러나 모든 경우 위처럼 다룬다면 당연히 메모리 낭비이다.
    그래서 변경이 잦은 string을 사용하고자 한다면,
    가변 string인 stringbuilder나 stringbuffer를 사용해야 한다.

#### 동기화 지원여부

    지원: StringBuffer
    미지원: StringBuilder

    동기화를 지원한다는 뜻은 멀티스레드에서 공유자원에 접근할 때 안전한지, 그렇지 않은지의 여부이다.
    StringBuilder는 동기화를 지원하지 않아 멀티스레드 환경에서 공유자원으로 활용된다면 위험할 수 있으나
    싱글스레드 환경에서 더 빠른 속도를 자랑한다.

## Spring 의 의존성 주입방식

    Spring 에서 의존성을 주입하는 방식에는 3가지가 있다.

    1. 필드 주입
    2. setter주입
    3. 생성자 주입

---

#### 예제

```java
@RestController
public class UserController {
    
}

@Service
public class UserService {

}
```

    위와 같은 두 클래스가 있다.

#### 1. 필드 주입

```java
@RestController
public class UserController {

    @Autowired
    private UserService userService;
}
```

    필드에서 @Autowired 어노테이션을 통해 주입하는 방식이다.
    간단하고 편리하나, 문제점은 class를 final로 지정할 수 없다는 점이다.

    mutable하여 잘못 수정하게 될 경우 NPE가 발생할 수 있다.

    그리고 필드 주입의 치명적인 문제점은, 자바만 이용하여 테스트코드를 작성할 수 없다는 점에 있다.
    자바 개발자라면 통합테스트와 단위테스트정도는 해보았을 것이다.

    통합테스트와 단위 테스트의 차이는, 통합 테스트는 Spring 위에서 돌아가고, 단위 테스트는 순수하게
    java testing framework (junit, mockito) 를 이용해서 이뤄진다는 점이다.
    그런데 필드주입을 하게되면 setter나 생성자로 mock객체나 기타 의존하는 객체를 주입할 수 없게 되므로
    단위테스트 작성시 반드시 NPE가 발생하게 된다.

#### 2. setter주입

```java
@RestController
public class UserController {
    
    private UserService userService;

    @Autowired
    public void setUserService(UserService userService) {
        this.userService = userService;
    }
}
```

    setter를 활용하여 주입받는다.
    필드 주입과 마찬가지의 문제점을 갖는다.

#### 3. 생성자 주입

```java
@RestController
public class UserController {

    private final UserService userService;

    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }
}
```

    생성자를 이용하여 주입한다.
    1. 필드 자체를 final로 지정할 수 있기 때문에, NPE문제에서 안전하다.
    -> 객체 생성 시점에 딱 한번만 초기화를 허용한다.

    2. 또한 스프링 자체적으로 생성자가 1개만 있을 경우 @Autowired를 생략해도 주입이 가능하도록 한다. (생략가능)

    3. 또한 단위테스트 작성이 용이할 뿐 아니라,
    
    4. 마지막으로 순환 참조 문제를 서버가 띄워질 때 방지할 수 있다.
    -> 이는 다른 주입방식과는 달리 생성자 주입이 스프링이 띄워질때 객체가 생성되어 Bean으로 등록됨과 동시에 DI를 하기 때문이다.
    A < - > B 양방향 참조를 하게되는 경우가 있다고 쳤을 때 Spring bean으로 등록될시
    new A(new B(new A(new B(....)))) 와 같이 무한참조가 발생하기 때문이다

    이와 반대로 나머지 주입방식은 생성 이후에 초기화되기 때문에, Runtime에 순환참조 문제가 발견된다 (서버 다운)
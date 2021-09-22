## Spring IOC (제어의 역전) 과 DI (의존성 주입)

    우선 결론부터 정리하자.
    IOC와 DI를 따로 이해하려고 하지 말자. IOC와 DI는 세트다. Spring이 IOC를 통해 DI를 하는 것이기 때문이다.
    
    IOC와 DI가 필요해진 배경은 이렇다.
    소프트웨어는 변화가 불가피하다. 요구사항은 시시때때로 변하기 때문이다.
    요구사항이 추가/변경 될 때마다 코드가 변경된다. 그런데 코드의 변경은 비용이다.
    많은 코드가 변경될수록 많은 비용이 든다.

    소프트웨어는 커지고, 의존관계의 복잡도는 올라간다.
    하나의 클래스를 수정해도, 해당 클래스에 의존하는 많은 클래스들이 선택과 생성의 책임을 갖기 때문에
    많은 코드가 수정된다.

    그래서 요구사항 변경시에도 코드가 최대한 변하지 않도록 프로그래밍 하는 것이 중요하다.

    어떻게 해야 소스코드의 변경을 최소화 할 수 있을까?
    스프링은 IOC와 DI로 그 해답을 찾았다.

    [장점]
    1. 낮은 결합도로 의존성을 줄인다.
    2. 테스팅에 유리하다.
    3. 요구사항 변화에도 영향을 최소화한다.

    아래 예시를 먼저보고, IOC와 DI를 이해하자.

### 예시

    시나리오

    문자열을 인코딩하는 프로그램을 만든다.
    Base64로 Encoding 하라는 요구사항이 있었다.

#### 1단계

```java
public class EncodingApplication {

    public static void main(String[] args) {
        String uri = "www.naver.com/books/it?page=10&size=20";

        String encoded = Base64.getEncoder().encodeToString(uri.getBytes());
        System.out.println("encoded = " + encoded);
    }
}
```

    위처럼 만들었다.
    그런데 여기서 Encoder 라는 클래스로 분리하여, 재사용성을 높일 수 있을 것 같았다.

#### 2단계

```java
public class Base64Encoder {

    public String encode(String string) {
        return Base64.getEncoder().encodeToString(string.getBytes());
    }
}

public class EncodingApplication {

    public static void main(String[] args) {
        String uri = "www.naver.com/books/it?page=10&size=20";

        Base64Encoder base64Encoder = new Base64Encoder();
        String encoded = base64Encoder.encode(uri);
        System.out.println("encoded = " + encoded);
    }
}
```

    아까전에 하나로 합쳐져 있던 코드를 위와 같이 클래스로 분리하여 재사용성을 높였다.

    여기서 유심히 봐야할 점은, 사용하는쪽 (EncodingApplication) 에서 직접 new 연산자를 통해
    Base64Encoder를 생성하고 활용하며 생명주기를 관리한다는 점이다.

    그런데 여기서 Base64 말고 UTF8로 Encoding 하라는 요구사항이 들어왔다.

    그래서 새로운 클래스를 만들었다.

```java
public class UTF8Encoder {

    public String encode(String string) {
        return URLEncoder.encode(string, StandardCharsets.UTF_8);
    }
}

public class EncodingApplication {

    public static void main(String[] args) {
        String uri = "www.naver.com/books/it?page=10&size=20";

        //아래 둘중 택일
        Base64Encoder base64Encoder = new Base64Encoder();
        String base64Encoded = base64Encoder.encode(uri);
        System.out.println("base64Encoded = " + base64Encoded);

        UTF8Encoder encoder = new UTF8Encoder();
        String utf8Encoded = encoder.encode(uri);
        System.out.println("utf8Encoded = " + utf8Encoded);
    }
}
```

    위처럼 만들었는데, 마찬가지로 new 연산자를 통해 생성하고 활용한다.
    Encoding이 변경될 때마다 의존하는 객체가 변경되고, 코드를 변경해야 하는 상황이다.

    그래서 추상화를 통해 조금 더 코드의 변경을 줄여보기로 했다.

```java
public interface IEncoder {
    String encode(String string);
}

public class EncodingApplication {

    public static void main(String[] args) {
        String uri = "www.naver.com/books/it?page=10&size=20";

        IEncoder base64Encoder = new Base64Encoder();
        String base64Encoded = base64Encoder.encode(uri);
        System.out.println("base64Encoded = " + base64Encoded);

        IEncoder utf8Encoder = new UTF8Encoder();
        String utf8Encoded = utf8Encoder.encode(uri);
        System.out.println("utf8Encoded = " + utf8Encoded);
    }
}
```

    사실상 추상화만 했을 뿐이지 거의 변화가 없다. 여전하다. 
    이제 추상화를 했으니, 다형성을 활용하기 위해 중간에서 IEncoder 객체들을 관리할 수 있는 클래스를 하나 만들기로 했다.

```java
public class Encoder {

    private IEncoder encoder;

    public Encoder() {
//        this.encoder = new Base64Encoder();
        this.encoder = new UTF8Encoder();
    }

    public String encode(String string) {
        return encoder.encode(string);
    }
}

public class EncodingApplication {

    public static void main(String[] args) {
        String uri = "www.naver.com/books/it?page=10&size=20";

        //UTF8로 인코딩하려면 Encoder 안의 코드를 변경해야 한다.
        Encoder encoder = new Encoder();
        String encoded = encoder.encode(uri);
        System.out.println("encoded = " + encoded);
    }
}
```

    이제 EncodingApplication의 코드는 하나로 합쳐졌다. 그리고 어떤 Encoding이냐에 따라서 Encoder만 수정하면 된다.
    하지만 이또한 요구사항 변경시마다 Encoder 클래스를 직접수정, 컴파일해야하는 상황이므로 
    "DI" 개념을 적용하여 사용하는 쪽에서 클래스를 갈아끼우기만 하면 되게 변경한다.

```java
public class Encoder {

    private final IEncoder encoder;

    public Encoder(IEncoder encoder) {
        this.encoder = encoder;
    }

    public String encode(String string) {
        return encoder.encode(string);
    }
}

public class EncodingApplication {

    public static void main(String[] args) {
        String uri = "www.naver.com/books/it?page=10&size=20";

        Encoder encoder = new Encoder(new Base64Encoder());
//        Encoder encoder = new Encoder(new UTF8Encoder());
        String encoded = encoder.encode(uri);
        System.out.println("encoded = " + encoded);
    }
}
```

    Encoder 외부에서 객체를 주입 (DI) 받도록 하니, 이제는 사용하는 쪽에서 알맞게 객체를 주입해주면 된다.
    추상화, 다형성, DI 를 적극적으로 활용하여 코드의 변경이 최소 (비용이 최소) 가 되는 코드를 만들어보았다.

    지금부터 Spring의 IOC와 결합한다.
    Springboot project와 결합해보자.

```java
@SpringBootApplication
public class EncodingApplication {

    public static void main(String[] args) {
        SpringApplication.run(EncodingApplication.class, args);
    }
}

@Component
public class ApplicationContextProvider implements ApplicationContextAware {

    private static ApplicationContext context;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        context = applicationContext;
    }

    public static ApplicationContext getContext() {
        return context;
    }
}
```

    Springboot project로 변경해주었다. 클래스는 그대로다.
    IOC를 통해 Spring Bean으로 관리해달라고 하기 위해 @Component 어노테이션을 붙여주었다.

    또한 Spring의 ApplicationContext에 직접 접근하여 Bean들을 주입받기 위해 Provider 클래스를 생성해주었다.

```java
@SpringBootApplication
public class EncodingApplication {

    public static void main(String[] args) {
        SpringApplication.run(EncodingApplication.class, args);
        ApplicationContext context = ApplicationContextProvider.getContext();


        String uri = "www.naver.com/books/it?page=10&size=20";
        
        Base64Encoder base64Encoder = context.getBean(Base64Encoder.class);
        Encoder encoder = new Encoder(base64Encoder);
        
        String encoded = encoder.encode(uri);
        System.out.println("encoded = " + encoded);
    }
}
```

    이제는 Spring Bean이 등록되어 있기 때문에 base64Encoder를 전과달리 new로 직접 생성해주지 않아도 된다.
    Spring에서 실행시점에 싱글톤으로 Bean으로 만들어 제공하기 때문이다.

    여기에 더해서 Encoder까지 Component로 등록해버렸다.

```java
@Component
public class Encoder {

    private IEncoder encoder;

    public Encoder(IEncoder encoder) {
        this.encoder = encoder;
    }

    public void setEncoder(IEncoder encoder) {
        this.encoder = encoder;
    }

    public String encode(String string) {
        return encoder.encode(string);
    }
}

@SpringBootApplication
public class EncodingApplication {

    public static void main(String[] args) {
        SpringApplication.run(EncodingApplication.class, args);
        ApplicationContext context = ApplicationContextProvider.getContext();

        String uri = "www.naver.com/books/it?page=10&size=20";

        Encoder encoder = context.getBean(Encoder.class);

        String encoded = encoder.encode(uri);
        System.out.println("encoded = " + encoded);
    }
}
```

    이제는 Encoder마저 Component로 등록했으므로 더이상 직접 new를 사용하지조차 않는다.

    마지막으로 Encoder에서 주입받는 IEncoder의 구현체가 여러개여서 발생하는 문제를 해결해보자.

```java
@Configuration
class IocConfig {

    @Bean
    public Encoder encoder(Base64Encoder base64Encoder) {
        return new Encoder(base64Encoder);
    }

//    @Bean
//    public Encoder encoder(UTF8Encoder utf8Encoder) {
//        return new Encoder(utf8Encoder);
//    }
}
```

    @Configuration - 여러개 Bean을 등록하겠다. 설정 파일이다.
    에 Bean을 등록해주고, 사용시 교체만 해주면 이제 코드를 전혀 건드리지 않아도 된다.

    만약 새로운 Encoder가 생긴다면, IEncoder를 상속받아 구현하고 위처럼 등록해주면 끝이다. 코드는 일절 수정하지 않아도 된다.

    또한 위의 과정은 xml 설정파일로도 동일하게 할 수 있다.

---

### IOC

    스프링 프레임워크를 사용해서 개발하면 개발자가 핵심객체의 생명주기를 직접 관리할 일이 없다.
    일반적인 POJO는 new 로 생성하여 관리하는데 비해 Spring framework를 이용하면
    객체가 Spring Container 안에 Singleton pattern으로 관리된다. 이를 Spring Bean 이라고 한다.

    객체의 생명주기 제어에 대한 권한이 개발자에서 Spring framework로 넘어갔음을 Spring IOC, 제어의 역전이라고 한다.

### DI

    IOC 로 인해 Spring Container에서 관리되는 Bean들을 적재 적소에서 사용할 수 있도록 제공하는 방법을 DI, 의존성 주입 이라고 한다.
    해당 객체가 필요한 시점에 Spring에서 singleton으로 관리되는 해당 객체를 주입하여 사용할 수 있도록 한다.

    참고로 Spring에서 DI 방식은
    1. 필드 주입 (@Autowired)
    2. setter 주입
    3. 생성자 주입
    이 있다.

---

## 정리

    위에서 코드의 유지보수성을 극대화하는 Spring의 IOC DI 과정을 보았다.
    
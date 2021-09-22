## Spring IOC (제어의 역전) 과 DI (의존성 주입)

    우선 결론부터 정리하자.
    IOC와 DI를 따로 이해하려고 하지 말자. IOC와 DI는 세트다. Spring이 IOC를 통해 DI를 하는 것이기 때문이다.
    
    IOC와 DI가 필요해진 배경부터 

### IOC

    스프링 프레임워크를 사용해서 개발하면 개발자가 핵심객체의 생명주기를 직접 관리할 일이 없다.
    일반적인 POJO는 new 로 생성하여 관리하는데 비해 Spring framework를 이용하면
    객체가 Spring Container 안에 Singleton pattern으로 관리된다. 이를 Spring Bean 이라고 한다.

    객체의 생명주기 제어에 대한 권한이 개발자에서 Spring framework로 넘어갔음을 Spring IOC, 제어의 역전이라고 한다.

### DI

    IOC 로 인해 Spring Container에서 관리되는 Bean들을 적재 적소에서 사용할 수 있도록 제공하는 방법을 DI, 의존성 주입 이라고 한다.
    해당 객체가 필요한 시점에 Spring에서 singleton으로 관리되는 해당 객체를 주입하여 사용할 수 있도록 한다.

    말로만 하면 잘 이해가 안될 것이다.

    일단 IOC & DI 를 이해하려면 의존을 이해해야 한다.

![image](https://user-images.githubusercontent.com/19279163/134355617-42e0d8e9-a6df-450e-9633-8c6a368d1f40.png)

    A가 B에게 의존한다는 것은, 
    B의 로직에 A 가 영향을 받는다는 말이고, A는 자신 뿐 아니라 B의 구현에 대한 책임도 갖는다.
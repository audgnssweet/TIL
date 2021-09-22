## Spring

    스프링 프레임워크는 자바 플랫폼을 위한 오픈 소스 애플리케이션 프레임워크로서 간단히 스프링이라고도 한다.

    간단히 말해서, Java를 이용해서 여러 어플리케이션을 만들 때 틀을 제공해주는 것이다.

### Spring의 핵심

    IOC (제어의 역전)
    DI (의존성 주입)
    AOP (관점 지향 프로그래밍)

    다른 프레임워크에는 없는 스프링만의 고유한 특징인 IOC와 DI
    스프링의 매우 큰 강점이다.

![image](https://user-images.githubusercontent.com/19279163/134351141-a82bc794-3bc6-41c8-8517-a2d323359764.png)

    스프링 삼각형

### Spring의 개발목적
    
    IOC, DI 를 통한

    1. 테스트의 용이성
    -> DI가 가능하기 때문에 mock객체를 주입가능
    -> 이전에는 외부 의존성 등을 직접 서버를 올려서 테스트하거나 했어야했는데 이제는 mock을 통해 수월하게 테스트

    2. 느슨한 결합
    -> 모듈간 결합도는 낮추고, 응집력은 높인다. 의존은 하지만 코드는 변하지 않는다.
    
### 대표적인 프로젝트 (모듈)

    Spring Boot
    기존 방대한 Spring framework에서 자주 사용하는 기본적인 설정들 혹은 WAS 설정 없이 바로 개발에 들어갈 수 있도록
    만든 프로젝트. 개발에 엄청난 편의를 제공한다.

    Spring data
    Spring framework를 이용해서 Database에 쉽게 접근할 수 있도록 하는 프로젝트. spring data jpa, spring data redis 등
    다양한 Database를 대상으로 프로젝트를 진행한다.

    Spring security
    Spring framework를 이용해서 Authentication, Authorization을 편리하게 관리할 수 있도록 하는 프로젝트로
    인증체계, OAuth등을 편리하게 이용할 수 있도록 지원한다.

    Spring Batch
    Spring framework를 이용해서 대용량 작업 (ex. 수십 수백만건의 데이터를 한꺼번에 건드리는 작업 등) 을 편리하게
    해결할 수 있도록 제공하는 프로젝트

    Spring cloud
    spring framework를 기존의 모놀리틱에서 요즘 화두인 MSA로 구성하는데 많은 도움을 주는 프로젝트

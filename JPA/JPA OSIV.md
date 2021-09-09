## JPA OSIV

    OSIV : open session in view

    JPA에는 영속성 컨텍스트라는 개념이 있다. 또한 이 영속성 컨텍스트의 생명주기는 트랜잭션과 같다.
    트랜잭션이 시작함과 동시에 영속성 컨텍스트가 생기고, 트랜잭션이 끝남과 동시에 영속성 컨텍스트도 사라진다.
    (원래는 그렇다)

    하지만 OSIV 설정을 하면 달라진다.
    OSIV는 영속성 컨텍스트를 Controller 및 view layer까지 가져갈 것인지를 설정하는 방식이다.

    설정을 먼저 보자.

```yaml
spring:
  jpa:
    open-in-view: true or false
```

    Spring boot 최신버전에서는 open-in-view 옵션이 기본적으로 true로 설정되어있다.
    
    이로 인한 효과는 아래와 같다.
    사용자의 request가 도달함과 동시에 영속성 컨텍스트가 시작된다. (트랜잭션은 시작되지 않는다.)

    그리고 controller -> service -> repository -> db 를 거쳐 다시 사용자에게 response가 나가는 시점 끝까지
    영속성 컨텍스트는 사라지지 않는다.

    (트랜잭션은 걸어둔 시점까지만 유지된다)

    그림으로 보면 아래와 같다.

![image](https://user-images.githubusercontent.com/19279163/132633959-0e0d9fca-73d3-4305-9dc7-87a06e4a54aa.png)

    JPA의 영속성 컨텍스트에서는 실제 객체를 key value 형태로 관리하기 때문에,
    영속화 되어있는 객체는 언제든지 DB를 거치지 않고 캐시처럼 바로 반환할 수 있다.

    위처럼 OSIV가 on 되었을 때의 장단점은 아래와 같다.

    장점
    1. Controller 단에서 지연로딩이 가능하다. 즉, DTO로의 변환이 Controller에서 일어나도 괜찮다.
    그로 인해서 서비스 로직에서는 Entity를 찾아서 반환해주는 로직까지만 들어가고 DTO로의 변환 책임은
    Controller로 분리할 수 있다.

    단점
    1. 영속성 컨텍스트를 가진다 == 데이터베이스 커넥션을 들고 있다.
    데이터베이스를 사용하기 위해서는, Connection Pool을 사용해야 한다.
    데이터베이스가 제공하는, 데이터베이스에 접근하기 위한 연결고리라 생각하면 된다.
    그런데 Connection pool의 개수는 제한적이다.

    최근 Hibernate에서 기본적으로 채택하는 Hikari pool은 디폴트로 10개정도 Connection pool를 가지는 것으로 알고있다.
    
    spring mvc는 동기로 동작한다. 
    만약 Controller에서 외부 API를 호출한다던지 해서 blocking 상태로 기다리면,
    그 시간동안 DB Connection이 낭비되는 것이다.
    
    그래서 실시간 트래픽이 매우 중요한 Service의 경우에는 OSIV를 설정했을 때 위험할 수 있다.

---

#### 언제 사용?

    Admin 처럼 사용빈도가 낮은 API를 개발할 때는 OSIV를 이용하면 복잡도도 낮추고 좋은 선택일 수 있다.
    그러나 실시간 트래픽이 매우 중요한 Service에서는 OSIV를 사용했다가 장애가 생길 수 있다.

---

#### OSIV를 끈 상태로 복잡성 관리하기

    핵심 Service에서 Lazy Loading을 처리하는 것은, Service의 복잡성을 늘릴 뿐 아니라 코드도 길어진다.

    그래서 
    1. 핵심 비즈니스 로직이 들어가는 Service와
    2. 한번에 많은 데이터로 복잡한 DTO를 만드는 Service
    로 분리하여 관리하는 것이
    복잡도를 낮추는데 좋은 선택일 수 있다.
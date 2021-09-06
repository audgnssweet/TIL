### H2 데이터베이스 설정하기

#### build.gradle 설정하기

```groovy
runtimeOnly 'com.h2database:h2'
```


#### application.yml 설정하기

```yaml
spring:
  h2:
    console:
      enabled: true

  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:testdb
    username: sa
    password:
```

    yml에 위와 같이 설정해주지 않아도 gradle 의존성만 있다면 H2 Console을 사용할 수 있지만,
    접속 버튼을 누르면 커넥션 에러가 뜨는 것을 확인할 수 있다.

    H2 에서는 데이터베이스에 접속하기 위해서는 미리 만들어주어야 하는데,
    위처럼 yml에 설정해주면 그런 과정을 거치지 않아도 된다.
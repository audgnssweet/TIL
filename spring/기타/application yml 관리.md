## springboot의 application yml 관리

    springboot에는 설정파일 대신 편리하게 사용할 수 있는 application.yml 파일이 있다.
    이는 여러개의 application-yml로 나누거나 혹은 profile 설정을 통해서 관리할 수 있다.
    또한 민감정보를 환경변수를 통해 관리하는 것도 가능하다.

---

### 환경변수 관리

```yaml
spring:
  datasource:
    url: ${DATASOURCE_URL}
    username: ${DATASOURCE_USERNAME}
    password: ${DATASOURCE_PASSWORD}
    driver-class-name: com.mysql.cj.jdbc.Driver
```

    보통 DB 관련 설정이나 AWS cloud 관련 설정시 민감한 정보를 포함하게 되는 경우가 많다.
    이런 경우 환경변수로 외부에서 값을 주입받는 방식을 선택할 수 있다.
    형식은 ${} 사이에 이름을 적고, 실제로 local pc 환경변수로 해당 이름의 환경변수를 넣어주거나
    혹은 intellij를 사용하고 있다면 intellij의 springboot 실행환경 설정을 통해서 환경변수를 넣어줄 수 있다.

    민감한 정보기 때문에 실제 배포환경에서는 해당 PC에 접속하여 환경변수를 세팅하거나 혹은 CD 툴을 통해서 세팅해준다.

#### 프로필 나누기

```yaml
spring:
  profiles:
    active: local
---
spring:
  config:
    activate:
      on-profile: local

  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: ${DATASOURCE_URL}
    username: ${DATASOURCE_USERNAME}
    password: ${DATASOURCE_PASSWORD}
---
spring:
  config:
    activate:
      on-profile: test

  datasource:
    url: jdbc:h2:mem:test
```

    위가 프로필을 나눈 상황이다. test 환경과 local 환경을 나눈 이유는 database이다.
    local 환경에서는 db를 mysql로 사용하고 싶고, test 환경에서는 db를 h2로 사용하고 싶어서이다.

    1. 프로필은 --- 세개를 통해서 나누고 spring.config.active.on-profile 를 통해서 프로필 이름을 지정한다.

    2. 실제로 사용할 profile은 spring.profiles.active로 설정한다.

    [추가]
    springboot가 실행될 때

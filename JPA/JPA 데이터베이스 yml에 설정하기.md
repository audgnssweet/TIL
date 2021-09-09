### JPA Hibernate 데이터베이스 설정하기

### application.yml 설정하기

#### H2 설정

```yaml
spring:
  h2:
    console:
      enabled: true

  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:~/{원하는 DB이름} 혹은  jdbc:h2:mem:testdb (일회성 메모리)
    username: sa
    password:

  jpa:
    show-sql: true
    open-in-view: true
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        dialect: org.hibernate.dialect.H2Dialect
        default_batch_fetch_size: 100
        format_sql: true
```

#### MySQL 설정

```yaml
spring:

  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: ${LOCAL_DB_URL}
    username: ${LOCAL_DB_USERNAME}
    password: ${LOCAL_DB_PASSWORD}

  jpa:
    show-sql: true
    open-in-view: true
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL5InnoDBDialect
        default_batch_fetch_size: 100
        format_sql: true
```

    datasource 에는 DB 연결 및 드라이버 관련 설정을,
    
    show-sql: SQL을 보일지

    open-in-view: OSIV를 적용할지

    ddl-auto: DDL문을 어떤 전략으로 실행할지 (create, create-drop, update, none)

    dialect: Database 방언 (매우중요)

    default_batch_fetch_size: Lazy Loading시 한번에 가져올 엔티티 수

    format_sql: SQL을 예쁘게
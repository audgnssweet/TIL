### JPA 쿼리를 콘솔에 보이게 하기

```yaml

spring:
  jpa:
    show-sql: true
    properties.hibernate:
      format_sql: true

```

    spring.jpa.show-sql 옵션이 JPQL을 콘솔에 띄우도록 하는 옵션
    spring.jpa.properties.hibernate.format_sql 옵션이 JPQL을 예쁘게 보이도록 하는 옵션
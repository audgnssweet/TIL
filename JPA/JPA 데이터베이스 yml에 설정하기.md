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

logging:
  level:
    org.hibernate:
      SQL: debug
      type: trace
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

    logging.level.org.hibernate.SQL: debug
    -> debug 레벨까지의 log로 출력한다.

    logging.level.org.hibernate.type: trace
    -> ? 값들을 출력해준다.

---

#### 쿼리를 보여주는 p6spy Library

```groovy
implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.7.1'
```

```java
public class P6spySqlFormatConfig implements MessageFormattingStrategy {

    @Override
    public String formatMessage(int connectionId, String now, long elapsed, String category, String prepared,
        String sql, String url) {
        sql = formatSql(category, sql);
        return now + "|" + elapsed + "ms|" + category + "|connection " + connectionId + "|" + P6Util.singleLine(prepared) + sql;
    }

    private String formatSql(String category, String sql) {
        if (sql == null || sql.trim().equals("")) return sql;

        if (Category.STATEMENT.getName().equals(category)) {
            String tmpsql = sql.trim().toLowerCase(Locale.ROOT);
            if (tmpsql.startsWith("create") || tmpsql.startsWith("alter") || tmpsql.startsWith("comment")) {
                sql = FormatStyle.DDL.getFormatter().format(sql);
            } else {
                sql = FormatStyle.BASIC.getFormatter().format(sql);
            }
            sql = "|\nHeFormatSql(P6Spy sql,Hibernate format):" + sql;
        }

        return sql;
    }
}

@Configuration
public class P6spyLogMessageFormatConfig {

    @PostConstruct
    public void setLogMessageFormat() {
        P6SpyOptions.getActiveInstance().setLogMessageFormat(P6spySqlFormatConfig.class.getName());
    }
}
```
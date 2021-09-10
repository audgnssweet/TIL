## Spring data JPA Auditing

    보통 비즈니스적 요구사항에서 쓰일 일이 많기 때문에
    등록, 수정, 등록자, 수정자 등은 테이블에 기본적으로 포함되는 경우가 많다.

    이런 것들을 매번 Entity를 DB에 저장할 때마다 하드코딩 해야한다면
    그야말로 반복적이고 지루한 코드가 될 가능성이 높다.

    JPA에서는 이를 AOP적 접근으로 깔끔하게 해결할 수 있도록 방법을 제공한다.

    해보자

---

#### Pure JPA로 사용하기

```java
@Getter
@MappedSuperclass
public class JpaBaseEntity {

    @Column(updatable = false)
    private LocalDateTime createdDate;

    private LocalDateTime updatedDate;

    @PrePersist
    public void prePersist() {
        LocalDateTime now = LocalDateTime.now();
        createdDate = now;
        updatedDate = now;
    }

    @PreUpdate
    public void preUpdate() {
        updatedDate = LocalDateTime.now();
    }
}
```

    먼저 @MappedSuperClass에 대해서 이해해야 한다.
    전에 JPA에서 상속을 통해 테이블을 구성해본 적이 있었다.
    그때처럼 상속을 통해서 필드를 내려받는 식으로 해결해야하는데 중요한 것은
    진짜 상속 관계가 되는 것이 아니라, JpaBaseEntity 안의 필드들만 필요한 것이다.
    이럴 때 사용하는 어노테이션이다.
    -> 필드들로 적용되는 상속임을 알리는 어노테이션이다.
    

    @PrePersist, @PostPersist, @PreUpdate, @PostUpdate

    네 가지 어노테이션이 있으니 적절한 것을 사용해주면 된다.
    어노테이션의 이름만 보아도 언제 실행될지 뻔한 것들이다.

#### Spring Data JPA로 사용하기

    Spring data JPA 에서 제공하는 어노테이션을 통해 사용하려면, 먼저 @EnableJpaAuditing 설정이 필요하다.

```java
@Configuration
@EnableJpaAuditing
public class JpaConfig {

}
```

    다음으로는 클래스를 보겠다.

```java
@EntityListeners(AuditingEntityListener.class)
@Getter
@MappedSuperclass
public abstract class BaseTimeEntity {

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime lastModifiedDate;

}
```

    @EntityListeners 어노테이션을 통해 JPA의 Auditing을 하겠다고 선언한다.

    @CreatedDate 는, 첫 Persist 시점에 자동으로 Timestamp를 생성해준다.
    
    @LastModifiedDate는, Entity의 Column이 변경될 때마다 자동으로 Timestamp를 생성해준다.

```java
@EntityListeners(AuditingEntityListener.class)
@Getter
@MappedSuperclass
public abstract class BaseEntity extends BaseTimeEntity {

    @CreatedBy
    @Column(updatable = false)
    private String createdBy;

    @LastModifiedBy
    private String lastModifiedBy;
}

@Configuration
@EnableJpaAuditing
public class JpaConfig {

    @Bean
    public AuditorAware<String> auditorProvider() {
        return "admin";
    }
}

```

    @CreatedBy 와 @LastModifiedBy 어노테이션을 통해 
    앞에서와 마찬가지의 시점에 자동 기록을 해주는데, 이 정보는 

    아래의 설정 클래스에서 정의해주어야 한다.

    AuditorAware을 반환해주면 되는데, 만약에 Security를 사용한다면
    여기에서 SecurityContext를 이용해서 정보를 넣어주던가,

    혹은 Session을 사용한다면 session을 사용해서 넣어주면 된다.

---

    [참고]
    Spring data JPA를 사용한다면, BaseEntity와 BaseTimeEntity 사이에도 상속 관계를 두어
    생성자 수정자는 선택적으로 적용할 수 있도록 하는 것도 좋은 방법이다.
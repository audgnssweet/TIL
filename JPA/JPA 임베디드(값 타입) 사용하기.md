## JPA 임베디드 (값 타입) 사용하기

    JPA에서 제공하는 강력한 기능중 하나인 값 타입을 사용하면,
    명확한 책임분리와 객체지향적 설계를 이룰 수 있다.

    회원 이름을 설계해보자.

---

#### Name (Embedded)

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Embeddable
public class Name {

    private String firstName;
    private String lastName;

    public Name(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    //==비즈니스 로직==//
    public String fullName() {
        return lastName + firstName;
    }
}
```

    @Embeddable 로 임베디드 값타입 객체임을 알린다.
    fullName을 얻는다던가 하는 비즈니스적인 로직도 정의하여 책임을 더욱 분산하는 구조를 만들 수 있다.

#### Member

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    @Embedded
    private Name name;
}
```

    @Embedded로 값타입임을 알린다.

---

    값 타입은 일반적인 JPA의 연관관계와는 달리, Member Table에 Column으로 들어간다.
## JPA 상속 단일테이블 전략 사용하기

    객체에는 상속 관계가 존재하지만, 데이터베이스 테이블에는 상속 관계가 없다.
    
    JPA의 상속을 이용하면
    상위 클래스에 공통 Column들을 정의해놓고 그것을 상속받아 사용할 수 있는 구조가 된다.
    아니면 상위 클래스 자료형으로 하위 클래스 모두를 가리킬 수 있는 다형성도 실현할 수 있다.
    또한 상위 클래스에 공통 비즈니스 로직을 몰아넣고 사용할 수 있다.
    (물론 객체지향에서와는 조금 다른 방법이다.)
    
    단일테이블 전략을 사용해보자 (한 테이블에 몰아넣기)

    Item을 Book과 Album이 상속받는 구조를 만들어보자.

---

#### Item

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "dtype")
@Getter
public abstract class Item {

    @Id
    @GeneratedValue
    @Column(name = "item_id")
    private Long id;

    private String name;

    private int price;

}
```

    @Inheritance 로 상속관계임을 표시하고, 전략을 단일 테이블 전략으로 선택한다.
    @DiscriminateColumn 으로 어떤 컬럼으로 식별 컬럼을 생성할지 정한다.
    
    또한 상위클래스는 pure하게 쓰이지 않을 가능성이 높으므로 abstract class로 지정해준다.

#### Book

```java
@Entity
@DiscriminatorValue(value = "B")
@Getter
public class Book extends Item {

    private String author;
    private String isbn;
}
```

#### Album

```java
@Entity
@DiscriminatorValue(value = "A")
@Getter
public class Album extends Item {

    private String artist;
    private String etc;
}
```

    위처럼 Item을 상속받고,
    @DiscriminatorValue 로 부모 클래스에서 정의한 식별 컬럼에 현재 타입은 어떤 값이 채워질지를 결정한다.


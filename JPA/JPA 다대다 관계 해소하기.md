## JPA 다대다 관계 해소하기

    JPA 다대다 관계 해소하기

    JPA에서 M:N은 1:N:1로 분리해야 한다.

    Item은 여러 카테고리에 속할 수 있고, Category는 여러 Item을 가질 수 있다.
    즉 다대다 관계라고 할 수 있다.

    이를 중간의 매핑 테이블인 CategoryItem을 두어 해결하면 된다.

---

#### Category

```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class Category {

    @Id @GeneratedValue
    @Column(name = "category_id")
    private Long id;

    private String name;
```

#### Item

```java
@Entity
@Getter
public class Item {

    @Id
    @GeneratedValue
    @Column(name = "item_id")
    private Long id;

    private String name;

    private int price;
```

### CategoryItem

```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class CategoryItem {

    @Id
    @GeneratedValue
    @Column(name = "category_item_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id")
    private Category category;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "item_id")
    private Item item;

}
```

    매핑 테이블을 두면, 중간 테이블에 외래키가 있어야 하므로 연관관계의 주인은 매핑 테이블이다.
    
    여기에 비즈니스적 요구사항 (EX. history 관리) 가 있으면 이 테이블에 넣어서 관리해주면 된다.
    
## JPA의 다대일 단방향과 양방향

    다대일 단방향과 양방향 설계해보기

    예제
    주문 (Order) 을 하면 주문상품 (OrderItem) 목록이 있다.
    즉 주문 1 : N 주문상품 이다.

---

#### OrderItem

```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class OrderItem {

    @Id
    @GeneratedValue
    @Column(name = "order_item_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id")
    private Order order;

    private String itemName;

    private int orderPrice; //상품 한개의 가격

    private int count;  //상품 수량
```

    일대다 연관관계에서는 연관관계의 주인이 무조건 다쪽이기 때문에,
    @ManyToOne 으로 다대일 연관관계임을 표시하고
    @JoinColumn으로 연관관계의 주인임을 표시한다.
    

#### Order 단방향

```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
@Table(name = "orders")
public class Order {

    @Id
    @GeneratedValue
    @Column(name = "order_id")
    private Long id;
    
    private LocalDateTime orderDate;

    @Enumerated(EnumType.STRING)
    private OrderStatus orderStatus;
```

    단뱡향이기에 아무런 처리가 필요없다.

#### Order 양방향

```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
@Table(name = "orders")
public class Order {

    @Id
    @GeneratedValue
    @Column(name = "order_id")
    private Long id;
    
    private LocalDateTime orderDate;

    @Enumerated(EnumType.STRING)
    private OrderStatus orderStatus;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private final List<OrderItem> orderItems = new ArrayList<>();
```

    @OneToMany로 일대다 연관관계임을 표시한다.
    mappedBy로 연관관계의 주인이 아님을 표시한다.
    컬렉션을 사용해서 일대다 관계를 관리한다.
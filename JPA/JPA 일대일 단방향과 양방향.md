## JPA의 일대일 단방향과 양방향

    일대일 단방향과 양방향 설계해보기

    예제
    주문 (Order) 과 배송 (Delivery) 는 일대일 관계이다.
    한 주문이 여러 배송을 가질수없고, 한 배송이 여러 주문을 가질 수 없다.

---

#### Order

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

    @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "delivery_id")
    private Delivery delivery;
```

    @OneToOne 어노테이션으로 일대일 관계임을 표시한다.
    @JoinColumn으로 연관관계의 주인임을 표시한다.

#### 단방향 Delivery

```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class Delivery {

    @Id
    @GeneratedValue
    private Long id;

    @Embedded
    private Address address;

    @Enumerated(EnumType.STRING)
    private DeliveryStatus deliveryStatus;
```

    단방향이기 때문에 아무런 처리도 해주지 않는다.

#### 양방향 Delivery

```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class Delivery {

    @Id
    @GeneratedValue
    private Long id;

    @OneToOne(mappedBy = "delivery")
    private Order order;

    @Embedded
    private Address address;

    @Enumerated(EnumType.STRING)
    private DeliveryStatus deliveryStatus;
```

    @OneToOne 으로 일대일 관계임을 표시하고,
    mappedBy 옵션으로 연관관계의 주인이 아님을 표시한다.
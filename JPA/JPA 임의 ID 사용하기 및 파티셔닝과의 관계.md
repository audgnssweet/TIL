## JPA 임의 ID 사용하기 및 파티셔닝과의 관계

    JPA에서 새로운 엔티티를 판별할 때 isNew 전략을 사용한다.
    그런데 GeneratedValue 외에 임의 ID를 사용한다면 무조건 DB를 한번 조회하고 영속화를 하게 됨을
    이미 알고있다.

    물론 이미 Long type의 GeneratedValue를 사용한다면 어지간한 데이터 record수는 전부 카바가 되지만,
    데이터가 정말 많을 것이라 예측하여 테이블을 파티셔닝한다던가 하는 경우에는 GeneratedValue 전략을
    사용할 수 없다.

    그럼 어떻게 할까?

    임의 ID를 사용하되, JPA에서 사용하는 isNew 메서드를 내가 정의해서 만들어주면 된다.

    해보자

```java
@EntityListeners(AuditingEntityListener.class)
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
@Entity
public class MyEntity implements Persistable<String> {

    @Id
    private String id;

    @CreatedDate
    private LocalDateTime createdDate;

    @Override
    public boolean isNew() {
        return createdDate == null;
    }
}
```

    Spring data에서 제공하는 Persistable interface를 implements 하여
    isNew를 Override해주면 된다.

    김영한 선생님은 createdDate == null 로 간편하게 처리하신다고 하는데,
    매우 좋은 방법인 것 같다.
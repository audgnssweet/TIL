## JPA 버전관리

    JPA에서 제공하는 낙관적 Lock을 사용하려면 버전 관리가 필요하다.

    @Version 어노테이션을 통해 관리하는데,
    Long, Integer, Short, Timestamp 에 적용할 수 있다.

```java
@Getter
@Table(name = "MEMBER")
@Entity
public class Member extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "ID")
    private long id;

    @Column(name = "NAME")
    private String username;

    @Column(name = "AGE")
    private int age;

    @Version
    private Integer version;
}

    @Test
    void test() {
        Optional<Member> byAge = memberRepository.findByAge(30);
        Member member = byAge.get();
        System.out.println(member.getVersion());

        member.updateName("업데이트");

        em.flush();
        em.clear();

        memberRepository.findByAge(30);
        System.out.println(member.getVersion());
    }
```

![image](https://user-images.githubusercontent.com/19279163/132939942-fd499751-b6f9-4b01-baf2-e78d734f8c51.png)
![image](https://user-images.githubusercontent.com/19279163/132939959-deaa5e8e-14e1-41c3-ad06-a4875836eff6.png)
![image](https://user-images.githubusercontent.com/19279163/132939963-149acfb5-5b42-4a04-8237-fcd04c10b426.png)

    위처럼 필드에 @Version 을 붙여주면
    자동으로 버전관리가 된다.
    처음에 영속화 되었을 때 0이었다가, 수정 될 때마다 1씩 증가한다.

    연관관계의 경우에는 각자의 필드를 수정하는 것으로는, 각자의 버전만 업데이트 된다.
    그러나 연관관계 필드 자체가 바뀌는경우 (FK) 에는 연관관계의 주인 버전만 업데이트 된다.

    또한 총 3번의 쿼리가 나갔음을 알 수 있는데, 주의해서 볼 것은
    두 번째 update 쿼리이다. 마지막에 where문에 version이 추가되었음을 알 수 있다.

    버전 관리를 시작하는 시점부터 자동으로 update시 버전을 확인하여, 만약에 트랜잭션 중간에
    다른 트랜잭션에서 수정하여 버전이 변경되었을 경우 버전 불일치로 에러를 터뜨린다.

    -> 두 번째 갱신 분실 문제를 없앨 수 있다.
    -> 최초 커밋만 인정하기 방식으로 없앨 수 있다.

#### 예제보기

```java
    @Transactional
    public void first() throws Exception {
        Optional<Member> found = memberRepository.findByAge(30);

        Thread.sleep(10000);

        found.get().updateName("first");
    }

    @Transactional
    public void second() {
        Optional<Member> found = memberRepository.findByAge(30);
        found.get().updateName("second");
    }
```

    두 메서드로 테스트를 해보았다.
    first를 먼저 호출하고, 그다음에 second를 호출하면
    first가 commit하기 전에 second가 commit해서 version이 변경된 상태가 된다.

    그렇다면 JPA에서 첫 커밋만 인정 방식으로 적용된다는 것을 확인해보자.

![image](https://user-images.githubusercontent.com/19279163/132940383-78fb2a92-63fc-42af-b675-858f2027cc7b.png)

![image](https://user-images.githubusercontent.com/19279163/132940389-06ea199c-4c4c-487e-b309-75abd38414b2.png)

![image](https://user-images.githubusercontent.com/19279163/132940397-47124b3f-3bf6-4273-9636-9989573e00ef.png)

![image](https://user-images.githubusercontent.com/19279163/132940421-fc729c43-3313-40e7-a929-655bcdd1bce1.png)

    우선 다른 트랜잭션에 의해서 데이터가 변경되었다는 에러가 잘 나오는 것을 확인했다.

    쿼리는
    first에서 select
    second에서 select 후 update
    first에서 update 하려 했으나, version이 맞지 않아 다시 select 해서 version확인 후 에러 발생

    순으로 잘 실행되었다.

---

    
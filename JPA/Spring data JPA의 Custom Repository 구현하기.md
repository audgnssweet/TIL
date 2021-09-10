## Spring data JPA의 Custom Repository 구현하기

    Spring data JPA가 많은 기능을 편리하게 제공한다고는 해도,
    개발자가 직접 Custom Repository 구현해야 할 일은 반드시 생긴다.

    바로 아래와 같은 경우이다.
    1. JPA직접사용 (EntityManager)
    2. JDBC 사용
    3. MyBatis 사용
    4. QueryDSL 사용

    위처럼 다른 DB관련 Library나 혹은 JPA를 직접 사용하면서, 동시에
    Spring data JPA에서 제공하는 JpaRepository의 효율까지 누리고 싶다면
    반드시 custom Repository를 구현해서 사용해야 한다.

---

#### 순서

    1. MemberRepositoryCustom 과 같이 인터페이스를 만든다.
    (이름은 상관없다. 안에 제공할 기능을 명시한다.)

    2. MemberRepositoryImpl 클래스를 만들고, 만든 Interface를 구현한다.
    (반드시 기존Repository에 Impl을 붙인 이름으로 해야한다.)

    3. MemberRepository가 MemberRepositoryCustom을 상속한다. (extends한다)

#### 예시

```java
public interface MemberRepositoryCustom {

    List<Member> findMemberCustom();
}

@RequiredArgsConstructor
public class MemberRepositoryImpl implements MemberRepositoryCustom{

    private final EntityManager em;

    @Override
    public List<Member> findMemberCustom() {
        return em.createQuery("select m from Member m", Member.class)
            .getResultList();
    }
}

public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom {

}

```

위 처럼 작성한 뒤

```java
    @Test
    void test() {
        memberRepository.findMemberCustom();
    }
```

![image](https://user-images.githubusercontent.com/19279163/132871111-d79091d0-d5ff-4e19-beef-d30a76a18a4c.png)

    이처럼 쿼리가 잘 나가는 것을 확인할 수 있다.

---

    [참고]
    repository custom이 능사는 아니다.
    관심사를 분리하여 오히려 repository를 여러개로 나누는 선택이 좋은 선택일 수 있다. (화면용 쿼리를 나눈다던지)
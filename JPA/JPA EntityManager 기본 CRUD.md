## JPA EntityManager 기본 CRUD

    JPA를 Spring data jpa와 함께 사용하지 않고 EntityManager만으로 사용하는 기본적인 방법

    기준 - Member Entity를 다루는 MemberRepository

#### 전체 코드

```java
@RequiredArgsConstructor
@Repository
public class MemberRepository {

    private final EntityManager em;

    public Long save(Member member) {
        em.persist(member);
        return member.getId();
    }

    public Member findOne(Long memberId) {
        return em.find(Member.class, memberId);
    }

    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
            .getResultList();
    }

    public List<Member> findAllPaging(int offset, int limit) {
        //order by id desc 하면 offset이 역순으로 적용됨에 주의
        List<Member> members = em.createQuery("select m from Member m", Member.class)
            .setFirstResult(offset)
            .setMaxResults(limit)
            .getResultList();

        Long count = em.createQuery("select count(m) from Member m", Long.class)
            .getSingleResult();

        return members;
    }

    public List<Member> findByName(String name) {
        return em.createQuery("select m from Member m where m.name = :name", Member.class)
            .setParameter("name", name)
            .getResultList();
    }

    public void delete(Member member) {
        em.remove(member);
    }
    
}
```

#### C (Create)

```java
    public Long save(Member member) {
        em.persist(member);
        return member.getId();
    }
```

    원래는 새로운 객체인지 판별하는 로직이 필요하지만, 간단하게 사용했다.

#### R (Read)

```java
    public Member findOne(Long memberId) {
        return em.find(Member.class, memberId);
    }
```

    단일 엔티티를 ID로 조회하는 방법이다.

```java
    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
            .getResultList();
    }
```

    단일 엔티티 조회 말고는, 조회 쿼리는 JPQL로 직접 작성해야 한다.

```java
    public List<Member> findAllPaging(int offset, int limit) {
        //order by id desc 하면 offset이 역순으로 적용됨에 주의
        List<Member> members = em.createQuery("select m from Member m", Member.class)
            .setFirstResult(offset)
            .setMaxResults(limit)
            .getResultList();

        Long count = em.createQuery("select count(m) from Member m", Long.class)
            .getSingleResult();

        return members;
    }
```

    페이징을 할 때는 위처럼 count 쿼리도 만들어서 사용한다.

```java
    public List<Member> findByName(String name) {
        return em.createQuery("select m from Member m where m.name = :name", Member.class)
            .setParameter("name", name)
            .getResultList();
    }
```

    JPQL에서 Parameter Binding을 처리하는 방식이다.
    번호로 매기는 Parameter Binding이 있고, 위처럼 Parameter name으로 하는 방식이 있는데,
    Parameter name 방식을 사용해야 한다.

#### U (update)

    JPA에서는 업데이트를 더티체킹을 통한 부분업데이트를 권장하므로,
    트랜잭션 안에서 엔티티를 조회하여 원하는 값을 변경해주는 것으로 충분하다.

#### D (delete)

```java
    public void delete(Member member) {
        em.remove(member);
    }
```
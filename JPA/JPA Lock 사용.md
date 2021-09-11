## JPA Lock 사용

    JPA에서 Lock을 사용해보자

```java
Member member = em.find(Member.class,1L,LockModeType.OPTIMISTIC);

em.lock(member,LockModeType.OPTIMISTIC);

JPA Repository를 사용한다면
@Lock(LockModeType.PESSIMISTIC_WRITE)
List<Member> findLockByUsername(String username);
```

    EntityManager로 조회 시점에 바로 Lock을 걸 수도 있고,
    찾은 엔티티로 중간 특정 시점에 Lock을 걸 수도 있고,
    
    Spring data JPA를 사용한다면 @Lock 어노테이션을 통해 Lock을 걸수도 있다.

## 낙관적 Lock (OPTIMISTIC)

    낙관적 Lock을 사용하려면 Entity에서 버전관리를 하고 있어야 한다.

    1. 기본적으로 버전관리가 들어가면, 
    격리 수준 READ_COMMITED 에 두 번째 갱신 분실 문제가 예방된다.

    2. 더하여 LockMode.OPTIMISTIC 을 걸어주면,
    앞에서와 달리 데이터의 수정이 없이 조회만 했을 때도 Version Check를 하여 
    트랜잭션이 종료되는 시점에 Version이 다르면 Exception을 발생시킨다.
    -> Dirty Read와 Non-repeatable Read를 방지한다.

    3. LockMode.FORCE_INCRENENT
    논리적인 단위의 엔티티 묶음의 버전 관리가 가능하다.
    예를 들어 게시글 1 : N 댓글 이라고 한다면
    댓글이 추가되거나 변경되더라도 글 자체의 Version은 증가하지 않는다.
    하지만 Version이 업데이트 된 것이나 다름 없다.
    이럴 때 LockMode를 걸어주면 트랜잭션 Commit 시점에 글의 Version도 증가한다.

## 
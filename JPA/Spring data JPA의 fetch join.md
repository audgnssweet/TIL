## Spring data JPA 의 fetch join

    DB에서 select시 데이터를 한꺼번에 끌어올 때 fetch join을 사용했다.

    Spring data JPA에서는 fetch join을 직접 쿼리로 사용하지 않고
    어노테이션을 이용해 지원한다.

```java
    @EntityGraph(attributePaths = {"team"})
    List<Member> findAll();

    @EntityGraph(attributePaths = {"team"})
    @Query("select m from Member m")
    List<Member> findAllWithQuery();

    @EntityGraph(attributePaths = {"team"})
    List<Member> findMemberByUsername(@Param("username") String username);
```

    1. @EntityGraph 어노테이션으로 fetch join을 지원한다.
    
    2. @Query로 JPQL을 정의함과 동시에 사용이 가능하다.

    [참고]
    아주 단순하게 fetch join할 때만 사용하는 것이 좋고,
    복잡해지면 QueryDSL로 해결하자.
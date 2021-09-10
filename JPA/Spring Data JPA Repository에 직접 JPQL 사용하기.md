## Spring Data JPA Repository에 직접 JPQL 사용하기

    Entity Manager를 통해서가 아니라 Spring Data JPA의 JPA Repository에 직접 JPQL을 사용할 수 있다.
    이는 컴파일 시점에 JPQL의 에러를 잡아주기에 좋다.

    복잡한 정적쿼리의 경우 직접 JPQL을 사용하여 편리하게 해결할 수 있다.
    그러나 QueryDSL을 사용하는 것도 좋다.

```java
    @Query("select m from Member m where m.username = :username and m.age = :age")
    List<Member> findUser(@Param("username") String username, @Param("age") int age);

    @Query("select m.username from Member m")
    List<String> findUsernames();

    @Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) from Member m join m.team t")
    List<MemberDto> findMemberDto();

    @Query("select m from Member m where m.username in :names")
    List<Member> findByNames(@Param("names") List<String> names);
```

    준비
    @Query 어노테이션을 사용하자

    1. 파라미터 바인딩

    이름 기반 파라미터 바인딩 방식을 사용하자. :age 처럼. 매우 직관적이다. 중간에 다른 파라미터가 추가되더라도 실수할 일이 없다.

    2. 반환 타입

    a. 엔티티 자체
    b. 엔티티의 특정 컬럼
    c. 엔티티 컬렉션
    d. 엔티티의 특정 컬럼 컬렉션
    
    참고로 리스트는 없는 경우에 null이 아니라 빈 collection을 반환한다.
    
    3. DTO로 바로 조회 가능

    단, 패키지 이름까지 넣어주어야 하는게 단점이다.
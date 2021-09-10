## Spring data JPA Paging에서 Count Query 분리하기

    Page<T> 사용하여 Paging시 자동으로 count Query가 나감을 확인할 수 있었다.

    그런데 조인이 여러번 들어가고, 조회쿼리가 복잡해지는 경우에는
    덩달아 count 쿼리도 복잡해져서 성능을 저하시키는 경우가 있다.

    이런 경우를 대비하여 count Query를 분리하는 방법을 제공한다.
    (실무에서는 분리하는 경우가 많다)

```java
    @Query(value = "select m from Member m left join m.team t",
        countQuery = "select count(m) from Member m")
    Page<Member> findByAge(int age, Pageable pageable);
```
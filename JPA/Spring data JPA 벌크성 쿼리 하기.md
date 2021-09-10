## Spring data JPA 벌크 쿼리

    벌크 쿼리란, 한꺼번에 엄청나게 많은 row에 동시에 적용될 logic을 처리하는 것을 말한다.
    JPA에서 이것을 하나하나 불러와서 처리하게 된다면, 영속성 컨텍스트가 꽉차서 비우고 채우고 비우고 하는
    작업을 반복하게 될 것이다. 그래서 DB에 직접 쿼리를 때려버리는 벌크 쿼리를 지원한다.

```java
    @Modifying(clearAutomatically = true)  //이 어노테이션이 있어야 jpa의 executeUpdate 를 실행한다.
    @Query("update Member m set m.age = m.age + 1 where m.age >= :age")
    int bulkAgePlus(@Param("age") int age);
```

    1. Member table 전체에 적용한다.

    2. 벌크 쿼리는 update로 시작한다.

    3. @Modifying 어노테이션을 통해 벌크 쿼리임을 알린다.
    (clearAutomatically = true) 옵션을 통해 영속성 컨텍스트를 자동으로 비워줄 수 있다.
    -> 벌크 쿼리는 DB에 직접적으로 쿼리를 쏘는 것이기 때문에, 같은 트랜잭션의 영속성 컨텍스트에는 반영이 안되어 있다.
    그러므로 꼭 사용 후에 영속성 컨텍스트를 비워주든지 아니면 뒤에 로직이 없고 트랜잭션을 종료하는게 좋다.
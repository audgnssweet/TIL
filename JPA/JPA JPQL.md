## JPA JPQL

    JPQL이란 엔티티 객체를 조회하는 객체지향 쿼리이다.
    
    JPQL은 SQL을 추상화해서 특정 DB에 의존하지 않고, Dialect를 설정해줌으로써
    코드를 변경하지 않고 데이터베이스를 수정하는 것이 가능하다.
    
    JPQL은 일반적인 SQL보다 간결하다.

---

    1. JPQL에서는 SELECT, UPDATE, DELETE를 지원한다.
    -> INSERT는 persist() 가 있으므로 필요가 없다.

    2. select from [where] [groupby] [having] [orderby]

    3. update [where]

    4. delete [where]

    5. JPQL 키워드는 대소문자를 구별하지 않는다. (엔티티나 필드는 한다)

    6. 별칭을 반드시 사용해야한다. 사용하지 않으면 에러가난다.
    -> select m from Member m

    7. JPQL 끝에 타입을 지정해주면 TypedQuery 이다. (형변환 필요 없음)
    -> em.createQuery("select m from Member m", Member.class);
    -> 지정해주지 않으면 Query 이고, Object로 반환하기 때문에 캐스팅이 필요하다.

    8. 결과가 한개이면 getSingleReulst() 를 사용한다.
    -> 만약 한개로 특정되지 않는다면 에러를 발생시킨다.

    9. 결과가 여러개이면 getResultList() 를 사용한다.
    -> 결과가 존재하지 않으면 빈 컬렉션을 반환한다.

    10. 이름 기반 파라미터 바인딩 방식을 사용해야 한다.
    -> em.createQuery("select m from Member m where m.username = :username", Member.class)
            .setParameter("username", "example");
    -> 파라미터 바인딩 방식은 선택이 아닌 필수다. 재사용성이 증가하고 공격으로부터 안전하다.

    11. Entity를 조회하면 영속성 컨텍스트에서 관리되고, 스칼라나 임베디드 값 타입을 조회하면
    영속성 컨텍스트에서 관리하지 않는다.

    12. Entity 이외 (임베디드 타입) 도 select 쿼리의 root가 될수는 없다.

    12. DISTINCT 를 통해 중복 데이터를 제거할 수 있다.
    -> select distinct m.username from Member m

    13. Object Projection이 아니라 DTO로 반환 받으려면 new 연산자를 사용해야 한다.
    -> select new (패키지).ExampleDto(m.id, m.username) from Member m

    14. 페이징 쿼리는 setFirstResult(), setMaxResults() 로 해야한다.
    -> em.createQuery("select m from Member m", Member.class)
            .setFirstResult(offset)
            .setMaxResults(limit);
    -> 보통 count 쿼리로 전체 count를 함께 반환한다.

    15. COUNT는 결과 수를 구한다. (Long 타입)
    -> select count(m) from Member m, Long.class
            .getSingleResult() 
    -> count 안에서 distinct를 사용 가능하다. (임베디드는 안됨)
    -> select count( distinct m.age) from Member m, Long.class 

    16. MAX, MIN 은 최대 최소 값을 구한다. (문자, 숫자, 날짜에 사용한다.)
    -> select max(m.age) from Member m,

    17. AVG 는 평균을 구하고, 숫자값에만 사용가능하다. (Double 타입)
    -> select avg(m.age) from Member m, Double.class
            .getSingleResult();

    18. SUM 은 합을 구한다. 숫자타입에만 사용 가능하다.
    -> 정수합은 Long, 소수합은 Double 등
    -> select sum(m.age) from Member m, Long.class
            .getSingleResult();

    19. 통계 함수에서 NULL 값은 무시한다.
    -> COUNT는 0이 된다.

    20. Group by 는 통계 데이터를 구할 때 특정 그룹끼리 묶어준다.
    -> select t.name, count(m.age) ... from Member m left join m.team t group by t.name
    -> 통계 쿼리를 구하는데 Member에서 left join 해서 team을 구해와서 team 이름으로 group by를 한 결과에다가
    특정 통계 쿼리를 적용해라.

    21. Having은 Group by와 함께 사용하고, 마지막에 필터링 할 때 사용한다.
    -> select t.name, count(m.age) ... from Member m left join m.team t group by t.name having avg(m.age) >= 10

    22. Order by는 정렬할 때 사용한다.
    -> order by (필드) (ASC|DESC)
    -> select m from Member m order by m.age asc, m.username desc

    23. join 은 기본적으로 inner join이다.
    -> select m from Member m (inner) join m.team t where t.name = :teamName
    -> 이런식으로 (m) 써주면 team의 id값만 가져온다.

    24. 외부 조인은 보통 left join을 사용한다.
    -> select m from Member m left (outer) join m.team t
    -> 단, 이런식으로 객체 자체를 (m) 써주면 team의 id값만 가져온다.

    25. fetch join은 JPA에서 성능 최적화를 위해 제공하는 join으로, 별도의 projection 없이도 join한 엔티티의
    모든 필드를 가져온다.
    -> 미리 땡겨오므로 지연로딩이 발생하지 않는다.
    -> select m from Member m [left] join fetch m.team t

    26. 일대다 조인은 결과가 뻥튀기되지만, 일대일, 다대일 조인은 아니다.

    26-1 연관관계 있는(FK) join (보통) vs. 연관관계 없는(일반칼럼) join
    -> 있는 : select m, t from Member m left join m.team t on ~~~
    -> 없는 카타시안 : select m, t from Member m, Team t
    -> 없는 외부조인 : select m, t from Member m left join Team t on ~~~

    26-2. inner join으로 해결이 되는 것은 inner join으로, 안되면 outer join으로 가자.
    -> join 중 성능이 가장 좋은 것이 inner join이다.

    27. 일대다 조인(컬렉션 조인)시 결과가 뻥튀기 될 때 결과를 getSingleResult() 로 받으면, 자동으로 필터링해서 한 개의 객체로
    만들어주지만, getResultList() 로 받으면 꼭 distinct를 붙여줘야 한다.

    28. 둘 이상의 컬렉션을 fetch join할 수 없다. (카타시안 곱 떄문에)

    29. 컬렉션을 fetch join하면 페이징이 불가능하다 (batch size로 해야한다.)

    30. x [not] between a and b (a,b 포함)

    31. x [not] in (컬렉션)

    32. 문자표현식 [not] like 패턴
    -> % 는 아무값 (없어도댐)
    -> _ (한글자 있어야하)
    -> select m from Member m where m.username like '_정%'

    
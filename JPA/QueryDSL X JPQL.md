#### 이 글은 인프런 김영한 선생님의 QueryDSL 강의를 보고 정리한 글입니다.

https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/dashboard

## QueryDSL X JPQL

    QueryDSL은 결국 JPQL을 쉽게 사용할 수 있도록 해주는 프로젝트이다.
   
    JPQL을 기반으로한 QueryDSL 사용법을 알아보자.

---

```java
        Member member1 = em.createQuery("select m from Member m where m.username = :username", Member.class)
            .setParameter("username", "멤버1")
            .getSingleResult();

        Member member2 = query.select(member)
            .from(member)
            .where(member.username.eq("멤버1"))
            .fetchOne();
```
    
    기본적인 search query이다.

```java
        Member foundMember = query.selectFrom(member)
            .where(
                member.username.eq("member1"),
                member.age.eq(10)
            )
            .fetchOne();

        List<Member> resultList = em.createQuery("select m from Member m where m.username = 'member1' and m.age = 10", Member.class)
            .getResultList();
    }
```

    where절에 search 조건을 , 로 이어서 여러개 넣을 수 있다.

```java
        QueryResults<Member> results = query.selectFrom(member)
            .fetchResults();

        em.createQuery("select count(m) from Member m", Long.class).getSingleResult();
        em.createQuery("select m from Member m", Member.class).getResultList();
```

    fetchResults() 를 호출하면, count쿼리까지 총 2개의 쿼리가 나간다.

```java
        long count = query.selectFrom(member)
            .fetchCount();

        em.createQuery("select count(m) from Member m", Long.class).getSingleResult();
```

    fetchCount() 를 하면 자동으로 count 쿼리로 변경해준다.

```java
        List<Member> res = query.selectFrom(member)
            .where(member.age.eq(100))
            .orderBy(member.age.desc(), member.username.asc().nullsLast())
            .fetch();

        em.createQuery("select m from Member m where m.age = 100 order by m.age desc, m.username asc nulls last",
                Member.class)
            .getResultList();
```

    orderBy로 정렬조건을 넣을 수 있다. 또한 여러개 조건을 나열하면 순서대로 적용된다.
    null이 어떻게 적용될지도 결정할 수 있다.

```java
        List<Member> page = query.selectFrom(member)
            .orderBy(member.username.desc())
            .offset(1)
            .limit(2)
            .fetch();

        em.createQuery("select m from Member m order by m.username desc")
            .setFirstResult(1)
            .setMaxResults(2)
            .getResultList();
```

    offset과 limit 도 넣어줄 수 있다. -> paging이 가능하다.

```java
        List<Tuple> res = query.select(
                member.count(),
                member.age.sum(),
                member.age.avg(),
                member.age.max(),
                member.age.min()
            )
            .from(member)
            .fetch();

        em.createQuery(
                "select count(m), sum(m.age), avg(m.age), max(m.age), min(m.age) from Member m")
            .getSingleResult();
```

    여러가지 통계 쿼리도 가능하다.

```java
        List<Tuple> res = query.select(team.name, member.age.avg())
            .from(member)
            .join(member.team, team)
            .groupBy(team.name)
            .having(member.age.avg().goe(20))
            .fetch();

        em.createQuery("select t.name, avg(m.age) from Member m join m.team t"
                + " group by t.name"
                + " having avg(m.age) >= 20")
            .getResultList();
```

    groupBy와 having으로 그루핑 및 필터링이 가능하다.

```java
        List<Member> members = query.selectFrom(member)
            .join(member.team, team)
            .leftJoin(member.team, team)
            .where(team.name.eq("team1"))
            .fetch();

        List<Member> res2 = em.createQuery("select m from Member m left join m.team t where t.name = :teamName",
                Member.class)
            .setParameter("teamName", "team1")
            .getResultList();
```

    join도 가능하다. 위는 PK 와 FK 로만 하는 연관관계가 있는 join이다.

```java
        List<Tuple> res = query.select(member, team)
            .from(member)
            .leftJoin(member.team, team).on(team.name.eq("팀1"))
            .fetch();

        em.createQuery("select m, t from Member m left join m.team t on t.name='팀1'")
            .getResultList();
```

    on 으로 join조건을 여러 개 나열하는 것이 가능하다.

```java
        List<Tuple> tuples = query.select(member, team)
            .from(member)
            .leftJoin(team).on(member.username.eq(team.name))
            .fetch();

        em.createQuery("select m, t from Member m left join Team t on m.username = t.name")
            .getResultList();
```

    연관관계 없는 join 도 가능하다. 이럴 때는 leftJoin() 안에 두 entity가 아니라 한 entity만 넣어준다. (outer join을 할)

```java
        Member foundMember = query.selectFrom(member)
            .join(member.team, team).fetchJoin()
            .where(member.username.eq("member1"))
            .fetchOne();

        em.createQuery("select m from Member m join fetch m.team t where m.username='member1'")
            .getSingleResult();
```

    fetch join이 가능하다.

```java
        List<Member> members = query.select(member)
            .from(member, team)
            .where(member.username.eq(team.name))
            .fetch();

        em.createQuery("select m, t from Member m, Team t");
```

    from절에 여러개를 넣어주면, thetajoin이 가능하다. 실제 쿼리는 cross join으로 나간다.
    실제 DB에서는 cross join으로 카테시안곱을 만들 수 있다.

```java
        QMember memberSub = new QMember("memberSub");
        Member maxAgeMember = query.selectFrom(member)
            .where(
                member.age.eq // in // goe 등 모두 가능(
                    JPAExpressions.select(memberSub.age.max())
                        .from(memberSub)
                )
            ).fetchOne();

        em.createQuery("select m from Member m where m.age = (select max(sm.age) from Member sm)", Member.class)
            .getSingleResult();
```

    where절 안에 subQuery를 넣는 것이 가능하다.
    이때는 JPAExpressions 를 사용해야 한다.
    또한 원래 DB에서와 같이 별칭이 달라야하는데, 이를 QClass를 하나 더 만들어줌으로써 해결해야 한다.

```java
        QMember subMember = new QMember("subMember");

        List<Tuple> fetch = query
            .select(
                member.username,
                JPAExpressions
                    .select(subMember.age.avg())
                    .from(subMember)
            ).from(member)
            .fetch();

        // nativeQuery = select m.username as un, (select avg(sm.age) from member as sm) as age_average from member as m;

        em.createQuery("select m.username, (select avg(sm.age) from Member sm) from Member m")
            .getResultList();
```

    select 안에 subQuery를 넣는 것도 가능하다.

    [추가]
    subQuery는 from절 안에는 불가능하다. JPA에서 아직 지원하지 않는다.

```java
        List<String> names = query
            .select(member.username)
            .from(member)
            .fetch();

        List<Tuple> tuples = query
            .select(member.username, member.age)
            .from(member)
            .fetch();

        List<MemberDto> memberDtos = query
            .select(Projections.bean(MemberDto.class,
                member.username,
                member.age))
            .from(member)
            .fetch();

        List<MemberDto> memberDtos = query
            .select(Projections.fields(MemberDto.class,
                member.username,
                member.age))
            .from(member)
            .fetch();

        List<MemberDto> memberDtos = query
            .select(Projections.constructor(MemberDto.class,
                member.username,
                member.age))
            .from(member)
            .fetch();

        QMember subMember = new QMember("subMember");

        query
            .select(Projections.fields(UserDto.class,
                member.username.as("name"),
                ExpressionUtils.as(
                    JPAExpressions
                        .select(subMember.age.max())
                        .from(subMember)
                    , "age")
            ))
            .from(member)
            .fetch();

        List<MemberProjectionDto> dtos = query
            .select(new QMemberProjectionDto(member.username, member.age)).distinct()
            .from(member)
            .fetch();
```

    Projection도 가능하다. JPQL에서는 new 연산자를 이용해야하는데,
    QueryDSL에서는 setter, 필드 직접주입, 생성자 주입, @QueryProjection 을 사용해서 주입 등의 방법을 제공한다.

```java
        long count = query
            .update(member)
            .set(member.username, "비회원")
            .where(member.age.lt(20))
            .execute();

        em.createQuery("update Member m set m.username='비회원' where m.age < 20").executeUpdate();

        long execute = query
            .update(member)
            .set(member.age, member.age.add(-1))
            .execute();

        long execute = query
            .update(member)
            .set(member.age, member.age.multiply(10))
            .execute();

        long execute = query
            .delete(member)
            .where(member.age.gt(20))
            .execute();

        em.createQuery("delete from Member m where m.age <= 20").executeUpdate();
```

    batch Query도 가능하다.
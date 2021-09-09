## JPA ToOne 조회쿼리 최적화 

    DB 성능문제의 거의 99%는 조회 쿼리에서 발생한다. CUD는 거의 단순한 경우가 많고, 한번에 수십개의
    CUD를 할 일이 잘 없기 때문이다. (있다면 보통 벌크쿼리로 처리할듯)

    MyBatis나 Spring JDBC Template를 사용하면 Query를 Native하게 가져갈 수 있고
    DBA가 다룬다면 정말 원하는 성능을 제대로 뽑아낼 수 있을지 모른다.
    
    하지만 JPA를 사용하면 생산성이 레벨이 다르다.
    또한 JPA에 대한 깊은 이해로 잘 사용만 한다면, 정말 답도없이 난해한 통계쿼리가 아니라면 거진 커버가 가능하다.

    실제로 국내 트래픽이 많은 IT 기업들에서 JPA를 사용하는 경우가 많은 것만 봐도 예측이 가능하다.

    조회쿼리를 최적화해보자.
    참고로 QueryDSL이나 Spring data JPA 를 사용하지 않은 Pure JPA이다.

---

    대상은 Article과 Comment로 하겠다.

    우선 결론부터 얘기하면 아래가 전부다.

    [전제] 모든 연관관계는 Lazy로 설정한다.
    ManyToOne 혹은 OneToOne : fetch join

    (기타) DTO를 통한 projection 연산, Native Query

    [참고] ToOne 관계에서 EAGER이 아니라 LAZY로 설정하는 이유는, EAGER로 했을 시에 엔티티 그래프를 따라서
    도대체 몇 개의 데이터를 한꺼번에 가져올지 예측이 안될 뿐 아니라, 원치않는 데이터도 항상 가져와지기 때문이다.

---

    ManyToOne 혹은 OneToOne 관계에서 쿼리를 줄여보자.

    상황은 이렇다. 게시글(Article)이 있고, 그 글을 작성한 회원(Member)이 있다.
    Article 정보를 가져와서 화면에 뿌려주는데, Member의 이름까지 글쓴이로 보여주어야해서, Member까지 끌어와야 하는 상황이다.

#### 기본적인 상황 (최적화 전)

```java
    @Repository
    public Article getOne(Long articleId) {
        return em.find(Article.class, articleId);
    }

    @Test
    void articleTest() {
        Article article = articleRepository.getOne(14L);
        Member member = article.getMember();
        System.out.println(member.getName());
    }
```

![image](https://user-images.githubusercontent.com/19279163/132690325-1d811512-cacb-4c4a-b822-57816893b5c1.png)

    코드는 위와 같다. 단순히 Article을 조회해서, 그 Article에서 Member를 조회하는 상황이다.
    어떻게 쿼리가 나갔는지 보여주는데, 쿼리가 총 2번 나갔음을 알 수 있다.

    먼저 Article 객체를 가져온 뒤에 Member 객체를 Lazy Loading하면서 DB에 쿼리를 날렸기 때문이다.
    
#### Fetch join 최적화

```java
    @Repository
    public Article getOne(Long articleId) {
        return em.createQuery(
                "select a from Article a"
                    + " join fetch a.member m"
                    + " where a.id = :articleId", Article.class)
            .setParameter("articleId", articleId)
            .getSingleResult();
    }

    @Test
    void articleTest() {
        Article article = articleRepository.getOne(14L);
        Member member = article.getMember();
        System.out.println(member.getName());
    }
```

![image](https://user-images.githubusercontent.com/19279163/132692708-f201d485-3c10-4754-b453-b52e663ffe8e.png)

    fetch join을 적용하니, 쿼리가 1개로 통합되었음을 알 수 있다.
    select를 유심히 보면 article을 가져올때 Member의 정보까지 처음부터 다 가져와버린다.

    그래서 Lazy Loading이 발생하지 않는 것이다.

    이게 지금은 뭐야 별차이 없잖아 라고 생각할 수 있지만, 연관관계가 복잡해지고
    여러 Entity들을 한꺼번에 로딩해야하는 상황이라면 1개 엔티티를 탐색할 때마다 쿼리가 1개씩 추가되므로
    엄청난 성능차이로 벌어질 수밖에 없다.

    참고로 ToOne관계에서는 그 개수가 몇개든 fetch join으로 가져와도 된다.


#### DTO 최적화

```java
@AllArgsConstructor
@Data
public class ArticleDto {

    private Long articleId;
    private String title;
    private String content;
    private String memberName;

}

    @Repository
    public ArticleDto getOne(Long articleId) {
        return em.createQuery(
                "select new jpa1.study.dto.ArticleDto(a.id, a.title, a.content, m.name) from Article a"
                    + " join a.member m"
                    + " where a.id = :articleId", ArticleDto.class)
            .setParameter("articleId", articleId)
            .getSingleResult();
    }

    @Test
    void articleTest() {
        ArticleDto dto = articleQueryRepository.getOne(14L);

        System.out.println(dto.getMemberName());
    }
```

![image](https://user-images.githubusercontent.com/19279163/132694216-c19c3080-c6ac-4ae1-9a37-a909f04c5cc5.png)

    위처럼 DTO로 바로 반환하도록 JPQL을 짜고 실행한다면
    쿼리에 보이듯이 필요한 Column만 Projection되어 자원을 아낄 수 있다.

    주의할 점은, Entity Graph에서 탐색하듯이 fetch join으로 하면 안된다는 점이다.
    fetch join은 select에 연관된 모든 Entity의 모든 column을 포함시키기 때문에 
    위처럼 Projection으로 가져올 Column을 제한한다면 에러가 생긴다.

    그리고 DTO로 바로 사용시 주의할 점은, 사실상 fetch join과 드라마틱한 성능차이가 있지는 않다는 것이다.
    한번에 가져올 column이 수십 수백개에서 줄어드는 것이라면 모르지만, 그정도가 아니라면 사실상 큰 차이는 없다.

    또한 DTO로 조회하는 것은 Client에 맞춘 로직 (핵심 비즈니스 로직이 아니다) 이기 때문에 사실상 잦은 변화가 생길
    가능성이 높다.

    그러므로 핵심 비즈니스용 Repository와는 분리하여, 관심사를 분리해서 사용해야 할 것이다.


#### 마지막

    위 방법들로도 해결이 안된다면 Native Query를 사용하는 방법밖에는 없다.
    Spring JDBC Template이나 JPA의 Native Query, 혹은 Mybatis를 사용하자.
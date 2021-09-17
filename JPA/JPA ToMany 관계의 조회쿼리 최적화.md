#### 이 글은 인프런 김영한 선생님의 JPA 활용2편을 듣고 정리한 글입니다.

https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-API%EA%B0%9C%EB%B0%9C-%EC%84%B1%EB%8A%A5%EC%B5%9C%EC%A0%81%ED%99%94/dashboard

## JPA ToMany 관계의 조회쿼리 최적화

    ## JPA ToMany 관계 조회쿼리 최적화 

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
    OneToMany
    페이징 없는경우 : fetch join, batch size
    페이징을 하는경우 : batch size

---

    OneToMany 관계에서 쿼리를 줄여보자.

    상황은 이렇다. 게시글(Article)이 있고, 그 글에 달린 댓글(Comment) 가 있다.
    Article 정보를 가져와서 화면에 뿌려주는데, 모든 댓글들과 댓글을 단 사용자의 정보까지 한꺼번에 가져와야 하는 상황이다.

    즉, article 1 : N comment 1 : 1 Member (작성자) 인 상황이다.
    1:N 연관관계가 있으면서 동시에 N+1 문제가 반드시 생기는 상황이다.

#### 기본적인 상황 (최적화 전)

```java
    @Repository
    public List<Article> findAll() {
        return em.createQuery("select a from Article a", Article.class)
        .getResultList();
    }

    @Test
    void articleTest() {
        List<Article> articles = articleRepository.findAll();
        for (Article article : articles) {
            for (Comment comment : article.getComments()) {
                System.out.println(comment.getContent() + comment.getMember().getName());
            }
        }
    }
```

![image](https://user-images.githubusercontent.com/19279163/132706035-b33d3a5d-f4ca-4505-9d02-bdd8349a9db4.png)
![image](https://user-images.githubusercontent.com/19279163/132706062-6da71e56-8cf0-4bf7-a6f2-1c9ac6906b93.png)
![image](https://user-images.githubusercontent.com/19279163/132706091-ccb5f661-b5a8-4ec7-9318-8906ee2bbf3f.png)
![image](https://user-images.githubusercontent.com/19279163/132706111-5d4c31ea-a643-42da-a9f6-b39f9d631db5.png)
![image](https://user-images.githubusercontent.com/19279163/132706126-c5c0ad64-bd74-4ecc-97b0-13d2b526de23.png)

    코드는 위와 같다. 단순히 Article을 조회해서, 그 Article에서 Comment를 조회하고, 그 Comment를 통해 Member 정보를 받아오는 방식이다.
    
    위에서 나간 쿼리를 보면, 가장 위의 article, 바로 아래 comments를 가져오는 쿼리 이후에
    Member정보를 가져오는 쿼리가 댓글 개수만큼 반복된다 (N + 1 문제 발생)

    글이 한개, 댓글이 3개여서
    글 1번, 댓글 1번, 댓글 3번의 쿼리로 가져오는 것이다.

#### 컬렉션 fetch join 최적화 (컬렉션이 포함되었기에 페이징 불가능)

    컬렉션 fetch join 최적화에는 단건조회, 다건조회가 있다.
    단건조회시에는 단순 fetch join만 해주면 되고, 다건조회의 경우에는 distinct를 붙여줘야 한다.
    이 distinct는 DB에 주는 distinct임과 동시에 JPA application 상에서의 distinct 명령이기도 하다.
    사실상 DB에서 row가 뻥튀기되는 문제는 distinct로 처리가 안되기때문에
    application에서 distict를 한다고 생각하면 된다.
    
    왜냐하면 결과가 em.find() 혹은 em.getSingleResult() 인 경우에는 1 에서 N 쪽으로 join했을 때
    데이터베이스에서 row가 뻥튀기 되는 현상을 JPA에서 알아서 잡아주는데
    다건조회의 경우에는 getResultList() 로 받기 때문에 뻥튀기 현상을 잡지 못하기 때문이다.

```java
    @Repository(단건조회)
    public Article findOne(Long articleId) {
        return em.createQuery("select a from Article a"
            + " join fetch a.comments c"
            + " join fetch c.member m"
            + " where a.id = :articleId", Article.class)
        .setParameter("articleId", articleId)
        .getSingleResult();
    }
    
    @Repository(다건조회)
    public List<Article> findAllFetch() {
        return em.createQuery("select distinct a from Article a"
            + " join fetch a.comments c"
            + " join fetch c.member m", Article.class)
        .getResultList();
    }

    @Test(단건조회)
    void findOne() {
        Article one = articleRepository.findOne(14L);
        for (Comment comment : one.getComments()) {
            System.out.println(comment.getMember().getName());
        }
    }

    @Test(다건조회)
    void findAllFetch() {
        List<Article> all = articleRepository.findAllFetch();
        for (Article article : all) {
            for (Comment comment : article.getComments()) {
                System.out.println(comment.getMember().getName());
            }
        }
    }
```

`단건조회`

![image](https://user-images.githubusercontent.com/19279163/132720129-b419e8d9-232b-426b-86d3-7b4f6be5cf06.png)


`다건조회`

![image](https://user-images.githubusercontent.com/19279163/132720160-ed0d48d1-4b74-4a47-b27e-2d499e85b930.png)

    두 코드 전부 조회 후 돌아가면서 모든 댓글의 모든 이름까지 전부 조회했는데도, 쿼리가 1번으로 끝났음을 알 수 있다.

    [주의] 컬렉션 fetch join으로 최적화한 다건 조회의 경우에는, 페이징이 불가능하다.
    왜냐하면 1 에서 N 으로 join이 들어가면, 데이터가 뻥튀기가 되기 때문이다.
    단건 조회의 경우에는 JPA 자체적으로 하나의 객체라고 단정지을 수 있기에 알아서 처리해주지만
    다건 조회의 경우에는 그게 불가능해서 쿼리에 distinct를 꼭 넣어주어야 했던 것이다.

    distinct 쓰면 제대로 가져오는데 페이징은 왜 안되냐?
    데이터베이스에서는 뻥튀기된 데이터를 그대로 주는데 application에서 distinct를 하기 때문이다.
    이게무슨말이냐면, DB에다가 offset이랑 limit을 날려서 데이터를 짤라서 받아야 하는데
    데이터가 뻥튀기 되기 때문에 그게 불가능하기 때문이다.

    만약에 메모리에서 하려면 뻥튀기된 모든 row를 가져와서 distinct 처리 한 후에 페이징을 해주면 되는데,
    뻥튀기된 row가 몇개나 될줄알고 이런 무모한 행위를 할수 없도록 JPA에서 자체적으로 막아놓은 것이다.
    그러니까 컬렉션 fetch join 다건조회를 할 수 있을 정도로 작은 데이터가 아니라
    페이징이 필요할정도로 많은 데이터의 경우에는 애초에 불가능하도록 해놓았다.

    [정리]
    단건조회일때는 컬렉션 fetch join을 적극적으로 활용하자
    다건조회일때는 컬렉션 fetch join을 활용하기보다는 batch size를 활용하자.

#### 컬렉션 batch size 최적화 (페이징 가능)

```yaml
spring.jpa.properties.hibernate.default_batch_fetch_size: 100
```

```java
    @Repository
    public List<Article> findBatch() {
        return em.createQuery("select a from Article a", Article.class)
            //.setFirstResult()
            //.setMaxResults()
            .getResultList();
    }

    @Test
    void findAllBatch() {
        List<Article> batch = articleRepository.findBatch();
        for (Article article : batch) {
            for (Comment comment : article.getComments()) {
                System.out.println(comment.getMember().getName());
            }
        }
    }
```

![image](https://user-images.githubusercontent.com/19279163/132726714-c7de5dcb-a1b3-415c-a7fd-6d8fb192b9fc.png)
![image](https://user-images.githubusercontent.com/19279163/132726732-8cb0147e-58e2-428e-bbda-84f4b9b51243.png)
![image](https://user-images.githubusercontent.com/19279163/132726756-a01f4d79-d6e4-4ddd-9d37-d911186d4fee.png)

    의도한대로 정확히 쿼리가 3번 나감을 알 수 있다.
    batch size는, 한번 Lazy Loading을 할 때 몇 개의 Proxy 객체를 실제 Entity로 긁어올지 결정하는 지표이다.
    그래서 해당 Class의 Entity Lazy Loading이 필요하다면, 영속성 컨텍스트에서 해당 자료형의 프록시 객체를 뒤져
    한번에 정해놓은 개수만큼 in query를 통해서 가져오는 것 같다.
    
    컬렉션을 fetch join 하는 것처럼 쿼리 1번으로 해결되는 것은 아니지만,
    페이징이 가능하다는 점에서 매우 실용적이고, 가장 좋은 방법이라 생각된다.

#### 기타

    이래도 해결이 안되는 경우에는 DTO보다는 차라리 Redis를 이용한 Caching을 채택하는 편이 좋을 것 같다.
    
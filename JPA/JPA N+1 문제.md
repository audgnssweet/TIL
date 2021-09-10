## JPA 의 N + 1 문제

    JPA에는 N + 1 문제가 있다.

    이를 이해하기 위해서는 먼저 Lazy Loading (지연 로딩) 에 대해 이해해야 한다.
    JPA에는 영속성 컨텍스트와 1차 캐시가 있다.
    Lazy Loading이란 연관관계가 있는 객체들을 DB에서 바로 조회해서 가져오는 것이 아니라
    영속성 컨텍스트에 해당 객체의 ID만을 가져와서 저장해두었다가 (프록시 객체)
    실제 해당 데이터의 사용 시점에 DB에 쿼리를 날려서 loading 하는 것을 의미한다.

    조회 쿼리를 1번 날려서 N개의 객체를 가져왔는데,
    그 N개의 객체의 연관관계 상세정보를 가져올 때 총 N번의 쿼리가 나가는 문제이다.

#### 예시

    글(Post), 댓글(Comment), 댓글 작성자(Member) 가 있다고 하자.

    연관관계는 이렇다.
    글 1 : N 댓글 N : 1 작성자

    그래서 로직은, 글의 PK를 통해 댓글 목록을 가져와서, 작성자 정보와 함께 띄워주는 것이다.

    한 게시글에서 모든 댓글을 조회하고 싶어서 댓글 목록을 가져왔다.
    그런데 화면에 정보를 띄워주려고 하니 댓글 작성자의 정보가 필요하다.

    현재 1개의 글에, 3개의 댓글이 달려있는 상태이다.
    만약에 N + 1 문제가 생긴다면,
    댓글의 작성자 정보를 보여주기 위해서 댓글 한개마다 댓글 작성자의 정보를 가져오기 위한 쿼리를 날린다.
    즉, 댓글 가져오는 쿼리 1번에 댓글 작성자 정보를 가져오는 쿼리 3번으로 총 4번의 쿼리가 나갈 것으로 예상된다.

    아래 코드와 결과를 보자.

```java

    @Repository
    public List<Comment> getComments(Long articleId) {
        return em.createQuery(
            "select c from Comment c"
            + " where c.article.id = :articleId", Comment.class)
        .setParameter("articleId", articleId)
        .getResultList();
    }


    @Test
    void nPlus1() {
        List<Comment> comments = articleCommentRepository.getComments(14L);
        for (Comment comment : comments) {
           System.out.println(comment.getContent() + "작성자: " + comment.getMember().getName());
        }
    }
```

![image](https://user-images.githubusercontent.com/19279163/132831197-cf3bd02d-a16e-4887-8087-763c097b233f.png)
![image](https://user-images.githubusercontent.com/19279163/132831061-53d16ebe-5467-48d4-9ee2-ffa062cf1631.png)
![image](https://user-images.githubusercontent.com/19279163/132831079-f18987d5-aeb4-43ca-9f26-205ed4178c9f.png)
![image](https://user-images.githubusercontent.com/19279163/132831096-da57d834-0f42-4ade-b753-723d12435897.png)

    예상대로 4번의 쿼리가 나간 것을 확인할 수 있다.
    이것이 N + 1 문제이다.
    만약에 댓글이 3개가 아니라, 수백 수천개였으면 쿼리가 수천개가 나가는 것이다.
    심각한 성능 장애로 이어질 수 있다.

---

## 해결법

    결론부터 말하면 fetch join과 batch size이다.
    이는 같은 폴더의 JPA ToOne관계 조회쿼리 최적화, JPA ToMany관계 조회쿼리 최적화를 참고하자.
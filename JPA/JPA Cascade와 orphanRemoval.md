# 📝 JPA의 CascadeType과 orphanRemoval에 대하여

    DDD 기반의 프로젝트를 진행하다가 자주 사용하게 되어 정리해본다.

## 🔎 Cascade

JPA에서 엔티티를 저장할 때, 해당 엔티티와 연관된 모든 엔티티는 영속 상태여야 한다.  
참고로 영속성 전이는 persist, remove를 호출할 때가 아니라 flush가 호출될 때 (트랜잭션이 끝날 때) 반영된다.  
-> 단, MySQL에서 identity로 기본 키를 사용하는 경우에는 save 호출시 바로 PK를 받아와야 하기 때문에 바로 호출된다.

### ❓ cascadeType.PERSIST

> PERSIST 를 이용하면 부모 엔티티만 영속 상태로 만들면 자식 엔티티도 영속 상태로 만들 수 있다.

```java
Board board = new Board();
Comment comment1 = new Comment();
Comment comment2 = new Comment();

//이 부분이 핵심으로, 부모에 대한 연관관계가 생성되기 때문에 한꺼번에 영속화 되는 것.
comment1.setBoard(board);
comment2.setBoard(board);

board.addComment(comment1);
board.addComment(comment2);

//board, comment1, comment2가 한꺼번에 영속화된다.
em.persist(board);
```

명심할 것은 단순히 엔티티를 영속화할 때 관련된 엔티티를 한꺼번에 같이 영속화하는 편리함만을 제공한다는 것이다.  
-> 연관관계를 추가한 다음 영속 상태로 만든다.

### ❓ cascadeType.REMOVE

> REMOVE 를 이용하면 부모 엔티티만 삭제하면 연관된 자식 엔티티를 모두 삭제할 수 있다.

```java
//해당 board와 연관된 모든 comment가 한꺼번에 삭제된다.
em.remove(board);
```

cascade가 설정되어있지 않은데 위와 같은 코드를 실행하는 경우, 부모만 삭제하려고 시도하며 이는 곧  
참조 무결성 제약조건 에러로 이어진다.

## 🔎 고아 객체 제거 (orphanRemoval)

> orphanRemoval을 사용하면
>
> 1번 기능 - '참조가 제거된 객체들' 을 삭제할 수 있다.  
> 2번 기능 - '부모가 삭제될 시 자식도 같이 삭제된다' (cascadeType.REMOVE 와 같음)

cascade와 마찬가지로 flush가 호출될 때 반영된다.  
중요한 것은 해당 엔티티로의 참조가 아예 없는 경우 삭제되기 때문에, 참조하는 곳이 반드시 한 곳일때만 사용해야 한다는 것이다.

---

## 👀 예시

## ✔ 유의점

- OneToMany 연관관계에 대해서만 적용된다.

  - 탐구 대상은 Many쪽의 객체이다.

- Many쪽의 Class의 JPA Repository를 실제로 호출하지 않는다.

  - 무조건 One쪽에서 Collection (Cascade와 orphan) 에 넣어줌으로써 함꼐 영속화 되는 경우만을 다룬다.

## 🎯 Board (글) 1 : N (댓글) Comment

### 📌 case 1

```java
class Board {

    @OneToMany(mappedBy = "board", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Comment> comments = new ArrayList<>();

    public void addComment(Comment comment) {
        comments.add(comment);
    }

    public void removeComment(Comment comment) {
        comments.remove(comment);
    }
}

class Comment {

    @ManyToOne
    @JoinColumn(name = "board_id")
    private Board board;
}
```

cascade ALL 과 orphanRemoval이 둘 다 걸려있는 경우이다.

1. addComment의 경우  
   매우 잘 동작한다. commentRepository.save() 호출해주지 않아도 잘 작동한다.
2. removeComment의 경우  
   매우 잘 동작한다. commentRepository.delete() 호출해주지 않아도 잘 작동한다.
3. boardRepository.delete(board) 의 경우  
   해당 글에 달려있던 댓글들이 한꺼번에 모두 지워진다.

### 📌 case 2

```java
class Board {

    @OneToMany(mappedBy = "board", cascade = CascadeType.ALL)
    private List<Comment> comments = new ArrayList<>();
}

class Comment {

    @ManyToOne
    @JoinColumn(name = "board_id")
    private Board board;
}
```

cascade ALL 만 걸려있는 경우이다.

1. addComment의 경우  
   매우 잘 동작한다. commentRepository.save() 호출해주지 않아도 잘 작동한다.
2. removeComment의 경우  
   동작하지 않는다. 명시적으로 commentRepository.delete() 를 호출해줘야 한다.
3. boardRepository.delete(board) 의 경우  
   해당 글에 달려있던 댓글들이 한꺼번에 모두 지워진다.

### 📌 case 3

```java
class Board {

    @OneToMany(mappedBy = "board", orphanRemoval = true)
    private List<Comment> comments = new ArrayList<>();
}

class Comment {

    @ManyToOne
    @JoinColumn(name = "board_id")
    private Board board;
}
```

orphanRemoval 만 걸려있는 경우이다.

1. addComment의 경우  
   동작하지 않는다. commentRepository.save() 를 명시적으로 호출해줘야 한다.
2. removeComment의 경우  
   잘 동작한다. commentRepository.delete() 를 호출해주지 않아도 된다.
3. boardRepository.delete(board) 의 경우  
   해당 글에 달려있던 댓글들이 한꺼번에 모두 지워진다.

# 👏 결론

### DDD시 적절하게 활용하자. 코드가 깔끔하고 직관적이게 된다.

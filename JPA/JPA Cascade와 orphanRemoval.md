# ๐ JPA์ CascadeType๊ณผ orphanRemoval์ ๋ํ์ฌ

    DDD ๊ธฐ๋ฐ์ ํ๋ก์ ํธ๋ฅผ ์งํํ๋ค๊ฐ ์์ฃผ ์ฌ์ฉํ๊ฒ ๋์ด ์ ๋ฆฌํด๋ณธ๋ค.

## ๐ Cascade

JPA์์ ์ํฐํฐ๋ฅผ ์ ์ฅํ  ๋, ํด๋น ์ํฐํฐ์ ์ฐ๊ด๋ ๋ชจ๋  ์ํฐํฐ๋ ์์ ์ํ์ฌ์ผ ํ๋ค.  
์ฐธ๊ณ ๋ก ์์์ฑ ์ ์ด๋ persist, remove๋ฅผ ํธ์ถํ  ๋๊ฐ ์๋๋ผ flush๊ฐ ํธ์ถ๋  ๋ (ํธ๋์ญ์์ด ๋๋  ๋) ๋ฐ์๋๋ค.  
-> ๋จ, MySQL์์ identity๋ก ๊ธฐ๋ณธ ํค๋ฅผ ์ฌ์ฉํ๋ ๊ฒฝ์ฐ์๋ save ํธ์ถ์ ๋ฐ๋ก PK๋ฅผ ๋ฐ์์์ผ ํ๊ธฐ ๋๋ฌธ์ ๋ฐ๋ก ํธ์ถ๋๋ค.

### โ cascadeType.PERSIST

> PERSIST ๋ฅผ ์ด์ฉํ๋ฉด ๋ถ๋ชจ ์ํฐํฐ๋ง ์์ ์ํ๋ก ๋ง๋ค๋ฉด ์์ ์ํฐํฐ๋ ์์ ์ํ๋ก ๋ง๋ค ์ ์๋ค.

```java
Board board = new Board();
Comment comment1 = new Comment();
Comment comment2 = new Comment();

//์ด ๋ถ๋ถ์ด ํต์ฌ์ผ๋ก, ๋ถ๋ชจ์ ๋ํ ์ฐ๊ด๊ด๊ณ๊ฐ ์์ฑ๋๊ธฐ ๋๋ฌธ์ ํ๊บผ๋ฒ์ ์์ํ ๋๋ ๊ฒ.
comment1.setBoard(board);
comment2.setBoard(board);

board.addComment(comment1);
board.addComment(comment2);

//board, comment1, comment2๊ฐ ํ๊บผ๋ฒ์ ์์ํ๋๋ค.
em.persist(board);
```

๋ช์ฌํ  ๊ฒ์ ๋จ์ํ ์ํฐํฐ๋ฅผ ์์ํํ  ๋ ๊ด๋ จ๋ ์ํฐํฐ๋ฅผ ํ๊บผ๋ฒ์ ๊ฐ์ด ์์ํํ๋ ํธ๋ฆฌํจ๋ง์ ์ ๊ณตํ๋ค๋ ๊ฒ์ด๋ค.  
-> ์ฐ๊ด๊ด๊ณ๋ฅผ ์ถ๊ฐํ ๋ค์ ์์ ์ํ๋ก ๋ง๋ ๋ค.

### โ cascadeType.REMOVE

> REMOVE ๋ฅผ ์ด์ฉํ๋ฉด ๋ถ๋ชจ ์ํฐํฐ๋ง ์ญ์ ํ๋ฉด ์ฐ๊ด๋ ์์ ์ํฐํฐ๋ฅผ ๋ชจ๋ ์ญ์ ํ  ์ ์๋ค.

```java
//ํด๋น board์ ์ฐ๊ด๋ ๋ชจ๋  comment๊ฐ ํ๊บผ๋ฒ์ ์ญ์ ๋๋ค.
em.remove(board);
```

cascade๊ฐ ์ค์ ๋์ด์์ง ์์๋ฐ ์์ ๊ฐ์ ์ฝ๋๋ฅผ ์คํํ๋ ๊ฒฝ์ฐ, ๋ถ๋ชจ๋ง ์ญ์ ํ๋ ค๊ณ  ์๋ํ๋ฉฐ ์ด๋ ๊ณง  
์ฐธ์กฐ ๋ฌด๊ฒฐ์ฑ ์ ์ฝ์กฐ๊ฑด ์๋ฌ๋ก ์ด์ด์ง๋ค.

## ๐ ๊ณ ์ ๊ฐ์ฒด ์ ๊ฑฐ (orphanRemoval)

> orphanRemoval์ ์ฌ์ฉํ๋ฉด
>
> 1๋ฒ ๊ธฐ๋ฅ - '์ฐธ์กฐ๊ฐ ์ ๊ฑฐ๋ ๊ฐ์ฒด๋ค' ์ ์ญ์ ํ  ์ ์๋ค.  
> 2๋ฒ ๊ธฐ๋ฅ - '๋ถ๋ชจ๊ฐ ์ญ์ ๋  ์ ์์๋ ๊ฐ์ด ์ญ์ ๋๋ค' (cascadeType.REMOVE ์ ๊ฐ์)

cascade์ ๋ง์ฐฌ๊ฐ์ง๋ก flush๊ฐ ํธ์ถ๋  ๋ ๋ฐ์๋๋ค.  
์ค์ํ ๊ฒ์ ํด๋น ์ํฐํฐ๋ก์ ์ฐธ์กฐ๊ฐ ์์ ์๋ ๊ฒฝ์ฐ ์ญ์ ๋๊ธฐ ๋๋ฌธ์, ์ฐธ์กฐํ๋ ๊ณณ์ด ๋ฐ๋์ ํ ๊ณณ์ผ๋๋ง ์ฌ์ฉํด์ผ ํ๋ค๋ ๊ฒ์ด๋ค.

---

## ๐ ์์

## โ ์ ์์ 

- OneToMany ์ฐ๊ด๊ด๊ณ์ ๋ํด์๋ง ์ ์ฉ๋๋ค.

  - ํ๊ตฌ ๋์์ Many์ชฝ์ ๊ฐ์ฒด์ด๋ค.

- Many์ชฝ์ Class์ JPA Repository๋ฅผ ์ค์ ๋ก ํธ์ถํ์ง ์๋๋ค.

  - ๋ฌด์กฐ๊ฑด One์ชฝ์์ Collection (Cascade์ orphan) ์ ๋ฃ์ด์ค์ผ๋ก์จ ํจ๊ผ ์์ํ ๋๋ ๊ฒฝ์ฐ๋ง์ ๋ค๋ฃฌ๋ค.

## ๐ฏ Board (๊ธ) 1 : N (๋๊ธ) Comment

### ๐ case 1

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

cascade ALL ๊ณผ orphanRemoval์ด ๋ ๋ค ๊ฑธ๋ ค์๋ ๊ฒฝ์ฐ์ด๋ค.

1. addComment์ ๊ฒฝ์ฐ  
   ๋งค์ฐ ์ ๋์ํ๋ค. commentRepository.save() ํธ์ถํด์ฃผ์ง ์์๋ ์ ์๋ํ๋ค.
2. removeComment์ ๊ฒฝ์ฐ  
   ๋งค์ฐ ์ ๋์ํ๋ค. commentRepository.delete() ํธ์ถํด์ฃผ์ง ์์๋ ์ ์๋ํ๋ค.
3. boardRepository.delete(board) ์ ๊ฒฝ์ฐ  
   ํด๋น ๊ธ์ ๋ฌ๋ ค์๋ ๋๊ธ๋ค์ด ํ๊บผ๋ฒ์ ๋ชจ๋ ์ง์์ง๋ค.

### ๐ case 2

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

cascade ALL ๋ง ๊ฑธ๋ ค์๋ ๊ฒฝ์ฐ์ด๋ค.

1. addComment์ ๊ฒฝ์ฐ  
   ๋งค์ฐ ์ ๋์ํ๋ค. commentRepository.save() ํธ์ถํด์ฃผ์ง ์์๋ ์ ์๋ํ๋ค.
2. removeComment์ ๊ฒฝ์ฐ  
   ๋์ํ์ง ์๋๋ค. ๋ช์์ ์ผ๋ก commentRepository.delete() ๋ฅผ ํธ์ถํด์ค์ผ ํ๋ค.
3. boardRepository.delete(board) ์ ๊ฒฝ์ฐ  
   ํด๋น ๊ธ์ ๋ฌ๋ ค์๋ ๋๊ธ๋ค์ด ํ๊บผ๋ฒ์ ๋ชจ๋ ์ง์์ง๋ค.

### ๐ case 3

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

orphanRemoval ๋ง ๊ฑธ๋ ค์๋ ๊ฒฝ์ฐ์ด๋ค.

1. addComment์ ๊ฒฝ์ฐ  
   ๋์ํ์ง ์๋๋ค. commentRepository.save() ๋ฅผ ๋ช์์ ์ผ๋ก ํธ์ถํด์ค์ผ ํ๋ค.
2. removeComment์ ๊ฒฝ์ฐ  
   ์ ๋์ํ๋ค. commentRepository.delete() ๋ฅผ ํธ์ถํด์ฃผ์ง ์์๋ ๋๋ค.
3. boardRepository.delete(board) ์ ๊ฒฝ์ฐ  
   ํด๋น ๊ธ์ ๋ฌ๋ ค์๋ ๋๊ธ๋ค์ด ํ๊บผ๋ฒ์ ๋ชจ๋ ์ง์์ง๋ค.

# ๐ ๊ฒฐ๋ก 

### DDD์ ์ ์ ํ๊ฒ ํ์ฉํ์. ์ฝ๋๊ฐ ๊น๋ํ๊ณ  ์ง๊ด์ ์ด๊ฒ ๋๋ค.

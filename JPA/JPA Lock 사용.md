## JPA Lock 사용

    JPA에서 Lock을 사용해보자

```java
Member member=em.find(Member.class,1L,LockModeType.OPTIMISTIC);

    em.lock(member,LockModeType.OPTIMISTIC);
    
    JPA Repository를 사용한다면
@Lock(LockModeType.PESSIMISTIC_WRITE)
List<Member> findLockByUsername(String username);
```

    EntityManager로 조회 시점에 바로 Lock을 걸 수도 있고,
    찾은 엔티티로 중간 특정 시점에 Lock을 걸 수도 있고,
    
    Spring data JPA를 사용한다면 @Lock 어노테이션을 통해 Lock을 걸수도 있다.

## 낙관적 Lock (OPTIMISTIC)

    낙관적 Lock을 사용하려면 Entity에서 버전관리를 하고 있어야 한다.

    1. 기본적으로 버전관리가 들어가면, 
    격리 수준 READ_COMMITED 에 두 번째 갱신 분실 문제가 예방된다.

    2. 더하여 LockMode.OPTIMISTIC 을 걸어주면,
    앞에서와 달리 데이터의 수정이 없이 조회만 했을 때도 Version Check를 하여 
    트랜잭션이 종료되는 시점에 Select쿼리를 통해 Version이 다르면 Exception을 발생시킨다.
    -> Dirty Read와 Non-repeatable Read를 방지한다.

    3. LockMode.FORCE_INCRENENT
    논리적인 단위의 엔티티 묶음의 버전 관리가 가능하다.
    예를 들어 게시글 1 : N 댓글 이라고 한다면
    댓글이 추가되거나 변경되더라도 글 자체의 Version은 증가하지 않는다.
    하지만 Version이 업데이트 된 것이나 다름 없다.
    이럴 때 LockMode를 걸어주면 트랜잭션 Commit 시점에 글의 Version도 증가한다.

#### 사용 예 (1. 버전관리만)

```java
@Transactional
public void first()throws Exception{
    Member member=memberRepository.findByAge(30).get();
    Thread.sleep(10000);

    member.updateName("first가 수정함");
}

@Transactional
public void second(){
    Member member=memberRepository.findByAge(30).get();
    member.updateName("second가수정함");
}
```

![image](https://user-images.githubusercontent.com/19279163/132981731-60972310-8786-48b7-b58d-f516b1e5778b.png)

![image](https://user-images.githubusercontent.com/19279163/132981742-bc986f7b-76fd-4576-8993-04a5bbdde88d.png)

![image](https://user-images.githubusercontent.com/19279163/132981763-0e6398f7-ff1b-4282-afa9-d155e086e467.png)
![image](https://user-images.githubusercontent.com/19279163/132981771-b46151de-5ab4-4092-9869-6ada587f82d6.png)

![image](https://user-images.githubusercontent.com/19279163/132981794-f202a14a-8820-4b9a-985f-622e7341468e.png)

    버전관리만 하는 경우에는 어플리케이션단에서 지원하는 Lock이고,
    데이터의 변경이 일어났을 때만 버전 체크를 하며,
    트랜잭션이 종료되는 시점에 충돌을 알 수 있게 되고,
    두 번째 갱신 분실 문제를 해결할 수 있다.

    과정은 이렇다.
    first() 요청으로, 먼저 특정 data를 읽어서 업데이트를 하는데, 10초의 딜레이를 준다.
    그 사이에 second() 요청으로 data를 변경해서 버전이 변경된 상황이다.
    에러가 잘 뜬 것을 확인할 수 있다.
    또한 그 결과값은 커밋을 먼저한 second() 의 결과값으로 잘 저장이 되어있는 것을 확인할 수 있다.

#### 사용 예 (2. OPTIMISITC_LOCK)

```java
    @Transactional
public void first()throws Exception{
    Member member=memberRepository.findByAge(30).get();
    Thread.sleep(10000);
}

@Transactional
public void second(){
    Member member=memberRepository.findByAge(30).get();
    member.updateName("second가 수정함");
}
```

![image](https://user-images.githubusercontent.com/19279163/132982916-5bce8c84-253d-4869-9f31-444b808e3e15.png)

![image](https://user-images.githubusercontent.com/19279163/132982920-7dad93d7-465b-4e30-bd58-5529a71c09b5.png)

![image](https://user-images.githubusercontent.com/19279163/132982928-c43fa531-3eaa-4ead-be0e-9561c7255e9a.png)
![image](https://user-images.githubusercontent.com/19279163/132982932-d6ba5f1a-6a1a-48bd-8932-2a698c95c316.png)

    OPTIMISITC Lock을 걸면 바로 위에서와의 차이점은
    first() 에서 읽기만 하는데도 트랜잭션이 commit 되는 시점에 버전 체크를 한다는 것이다.
    
    이를 통해서 repeatable read 까지도 구현할 수 있다.

    실행 결과 쿼리를 보면
    first() 에서 먼저 select를 하고, second() 에서 select 후 update를 한다.
    이후에 commit시점에 first() 에서 select를 했는데, 존재하지 않아서
    JPA에서 버전체크용으로 한번 더 select 후 없다는것이 확인되고 에러를 터뜨린다.

## 비관적 Lock (PESSIMISTIC)

    비관적 Lock은 데이터베이스 레벨에서 제공하는 Lock을 의미한다.
    주로 select for update 구문을 사용하면서 시작하며 Lock을 건다.
    주로 PESSIMISTIC_WRITE 모드를 사용한다.

#### 사용예 (PESSIMISTIC_WRITE_LOCK)

```java
    @Transactional
public void first()throws Exception{
    Member member=memberRepository.findLockByAge(30).get();
    Thread.sleep(10000);

    member.updateName("first가 수정함");
}

@Transactional
public void second(){
    Member member=memberRepository.findByAge(30).get();
    member.updateName("second가 수정함");
}
```

![image](https://user-images.githubusercontent.com/19279163/132983212-4baddecb-28bc-4a82-bcde-0521e1a71750.png)

![image](https://user-images.githubusercontent.com/19279163/132983222-b87da029-96f9-4a4c-b657-d08a3d365393.png)

![image](https://user-images.githubusercontent.com/19279163/132983242-321737b4-dd6d-46e9-bc61-da995e196545.png)
![image](https://user-images.githubusercontent.com/19279163/132983250-0bc69d52-c754-4ffd-9fd3-8badcbab055d.png)

![image](https://user-images.githubusercontent.com/19279163/132983291-66618448-ef88-4c1c-a484-44f0af9f088b.png)

    비관적 Lock은 DB에 거는 Lock이므로 second() 가 commit하는 시점에 바로 에러가 발생한다.

    비관적 Lock에서 중요한 것은 누가 먼저 Lock을 걸었냐인데,
    first()에서 먼저 Lock을 거므로, 에러가 터지고 결과적으로 first()의 수정이 들어가야 한다.
    이것도 마찬가지로 잘 들어갔음을 알 수 있다.

#### 정리

    Database Lock에 대해서 알아보았다.
    중요한 것은, 애플리케이션 레벨의 Transactional(DB Connection) 과 실제 DB의 트랜잭션은 다르다는것을 이해해야 한다.
    그래서 MySQL 에서 default로 repeatable read를 지원하지만 어플리케이션상에서 영속성 컨텍스트를 날리고
    테스트해보면 repeatable read가 되지 않는다.
    
    결국 repeatable read를 디폴트로 된다고 생각하면 안되고,
    OPTIMISTIC_LOCK 이나
    PESSIMISTIC_WRITE 이나
    혹은 영속성컨텍스트를 초기화시키지 않고 그냥 사용함으로써 구현해야 한다.
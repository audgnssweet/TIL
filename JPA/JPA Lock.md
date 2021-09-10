## JPA Lock

    Lock을 하는 이유는, 트랜잭션의 격리 수준을 선택하기 위함이다.
    격리 수준과 동시성 사이에서 저울질을 잘 해야한다.

    JPA에서 제공하는 Lock에는 2가지 종류가 있다.
    
    1. 낙관적 Lock
    2. 비관적 Lock

    알아보자.

#### 낙관적 Lock (OPTIMISTIC_LOCK)

#### 비관적 Lock (PESSIMISTIC_LOCK)
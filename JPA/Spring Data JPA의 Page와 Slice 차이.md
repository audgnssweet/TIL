## Spring Data JPA의 Page와 Slice 차이

    Spring data JPA에서 제공하는 Paging 객체는 Page<T> 와 Slice<T> 가 있다.
    둘의 차이를 비교해보자.

#### Page<T>

```java
    @Test
    void test(){
        Page<Member> page=memberRepository.findAll(
            PageRequest.of(1,10)
        );
    }
```

![image](https://user-images.githubusercontent.com/19279163/132845970-ddfc5616-ad10-4ba4-a1b5-46d4d3dd0e57.png)
![image](https://user-images.githubusercontent.com/19279163/132846268-4c344a97-f8c4-4277-8ce8-71a2bab71db9.png)

    1. 쿼리를 2번 날린다.
    데이터의 총 개수를 전달해주기 위해서 count쿼리까지 총 2번의 쿼리를 날린다.

    2. Page객체의 구성
    content 이외에도, totalCount, sort, page, size 등 다양한 정보가 들어있다.

#### Slice<T>

```java
    Slice<Member> findAllByAge(int age, Pageable pageable);
```

![image](https://user-images.githubusercontent.com/19279163/132846820-cb4eaa76-2a82-4fdc-a794-7c8a706764ce.png)
![image](https://user-images.githubusercontent.com/19279163/132846952-26e98858-1a7c-4924-bf61-993345073c3a.png)

    1. 쿼리를 1번 날린다.

    대신 limit에서 원래 지정해준 limit보다 + 1 해서 쿼리를 날려서, 결과가 있으면 hasNext를 채워준다.

---

#### 정리

    결론을 정리하면,

    웹사이트 페이징에는 Page<T> 를 사용하고
    모바일 환경에서는 Slice<T> 를 사용하자.
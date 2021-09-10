## Spring Data JPA의 Query Method

    Spring data JPA에서는 기본적인 쿼리들
    ex) save, findById, findAll, delete 등
    을 미리 구현해놓고, 인터페이스만 상속받으면 사용할 수 있도록 해준다.

    그런데 실제 비즈니스에서는 이런 간단한 조회 말고 해당 도메인에 특화된 여러 쿼리가 필요하다.
    그런데 개발자들이 Spring Data JPA에서 제공하는 기능도 사용하면서 도메인에 특화된 기능을 직접 구현하려고 하니 
    Spring Data JPA에서 제공하는 Repository의 메서드를 모두 구현해야하는 대참사가 발생해서 이를 해결하고자 Query Method를 도입했다.
    (클래스에서 구현해야하는데 implements 하면 전부 구현해야하니까. JPA Repository는 Interface만으로 사용할 수 있다.)

    매우 강력한 기능이니 잘 알아두면 좋다.

    [참고]
    강력하고 편리한 기능이지만, 조건이 2개가 넘어가고 복잡해지면 함수 이름이 너무 길어지는 단점이 있다.
    그래서 조건이 여러개인 복잡한 쿼리는 JPQL을 사용하거나 QueryDSL로 처리하자.

---

    Query Method는 Method의 이름만 잘 지어주면, Spring Data JPA에서 그에 맞춰서 JPQL을 생성해주는 아주 강력한 기능이다.

#### 예시

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    Optional<Member> findByAge(int age);
}

    @Test
    void test() {
        memberRepository.findByAge(25);
    }
```

![image](https://user-images.githubusercontent.com/19279163/132840720-0261b26b-fac5-4f3e-8fe4-572265ca01c7.png)

    위처럼 쿼리가 잘 생성되어 나감을 확인할 수 있다.

## 공식문서

    spring.io 에서 Spring Data Project의 Official Docs를 참고하면 된다.

![image](https://user-images.githubusercontent.com/19279163/132840915-308293ec-cae6-4e2f-aaba-5b9507b47e90.png)
![image](https://user-images.githubusercontent.com/19279163/132840941-27ffa67d-3d8f-4be9-b7a3-0681cd141b2f.png)


    In에는 Collection을 넘겨주어 속하는지 아닌지를 판단할 수 있다.
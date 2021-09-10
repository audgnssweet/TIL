## Spring Data JPA의 Repository 상속효과

    Spring Data JPA의 Repository인

    JpaRepository, PagingAndSortingRepository, CrudRepository를 상속받으면
    Runtime시에 Spring Data 에서 알아서 구현체를 만들어서 Bean으로 주입해준다.

    @Repository를 붙이지 않아도 자동으로 Component Scan이 되어 Spring Bean으로 등록 된다.

    마지막으로
    JPA 예외 (DB에 밀접한 예외) 를 Spring 에서 일괄적으로 처리할 수 있도록
    Spring에서 추상화한 예외 형식으로 변환해주는 역할을 한다.
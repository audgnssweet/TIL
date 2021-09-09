## API Response로 List를 넘기는 것

    API Response로 List를 넘기는 것은 자제해야한다.
    왜냐하면 어떤 비즈니스적 요구사항이 List에 얹어질지 모르기 때문이다.
    
    List를 다른 Response 객체로 한번 감싼다면, 추후에도 field하나 추가하는 정도의
    비용만 요구될 뿐이다.

#### 비슷한 상황

    마치 JPA에서 다대다 연관관계를 @JoinTable 을 통해 해결해서는 안되는 것과 일맥상통한다.

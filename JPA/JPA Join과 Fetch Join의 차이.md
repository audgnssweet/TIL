## JPA Join과 Fetch Join의 차이

    Join은 일반적인 데이터베이스 테이블을 대상으로 하는 Join이다.
    그래서 select절에 원하는 column 값을 넣어주지 않으면 column을 select해오지 않는다.
    또한 join으로 컬럼을 명시하게 된다면 단일 Entity 자료형으로 넣는 것이 불가능하다. (DTO 말고)

    반면에 fetch join은 객체를 대상으로 하는 join이라고 생각하면 된다.
    별다른 column 명을 명시해주지 않아도, fetch join의 대상이되는 entity의
    모든 필드를 자동으로 select절에 추가해준다.
    그리고 fetch join해서 가져온 필드를 적절하게 잘라서 대상 객체의 연관관계에 맞게 넣어준다.
    그래서 단일 자료형으로 받는 것이 가능하다. (엔티티 그래프탐색 가능)

    그러므로 DTO로 바로 반환하는 경우 (projection 하는 경우) 에만 사용하고
    entity graph 탐색이 목적이라면 fetch join을 사용하자.

    참고로 fetch join의 select 생성 특성때문에,
    DTO로 바로 반환해서 projection 연산을 하는 경우에는 fetch join을 사용하면 에러를 뱉는다.
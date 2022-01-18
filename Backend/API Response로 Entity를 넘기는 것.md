## API Response로 Entity를 넘기는 것

    자제해야 한다.
    Entity에는 많은 정보가 포함되어있는데, 원치 않는 정보까지 노출될 우려가 있을 뿐 아니라
    Entity 자체가 API의 스펙이 되어버리기 때문에 Entity가 변경되면 연관된 API 스펙이 전부 변경되어 버린다.

    또한 Entity에는 연관관계 필드가 많은데, 무한 참조가 발생하거나 필드가 null로 채워져서 보내질 가능성이 높다.

    마지막으로 Embedded 값 타입은, immutable pojo이기 때문에 그냥 넘겨도 괜찮다.

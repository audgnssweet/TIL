## JPA 엔티티 설계원칙

---

#### 연관관계는 가급적 단방향을 사용하자
    
    반대 방향에서 참조하는 경우가 많다면, 양방향을 고려해도 되지만,
    일반적으로 그럴 필요가 없는 상황에서 양방향으로 가져가는것은 오히려 복잡도를 올린다.

#### M:N 은, M : 1 : N 으로 풀어라.

    M : N 연관관계의 경우 매핑 테이블을 하나 두는 것이 좋다.
    관리도 쉽고, 이력 추적이나, 새로운 비즈니스 요구사항들이 생겼을 때 테이블에 쉽게 추가하여 관리할 수 있다.

#### FK는 가급적 optional = false 옵션을 사용해라. - 연관관계의 주인 (비즈니스적 요구사항이 있지 않는 경우)

    JPA의 기본 JOIN 전략은 OUTER JOIN이다.
    그런데 성능면에서는 당연히 INNER JOIN이 좋다. 변경해주자.

    ToOne 관계의 연관관계의 주인 필드에서는 두 가지 어노테이션이 반드시 붙는다.
    @ManyToOne or OneToOne(optional=false)
    @JoinColumn(nullable=false)
    
    null을 허용할지, 그렇지 않을지는 위처럼 세팅할 수 있는데
    이때 둘중 하나를 설정해주면 된다.
    그런데 optional=false 옵션은 where절에 외래키 사용시에도 inner join을 실행하기 때문에 권장한다.

#### OneToOne 연관관계에서 연관관계의 주인은, 더 많이 참조하는 쪽에서 해라.

    말 그대로다.

#### 가급적 Entity에 Setter를 두지 말고 변경용 비즈니스 메서드를 사용해라.

    Setter의 무분별한 사용은 객체가 어느 시점에 어떤 목적으로 변경되었는지를 트래킹하기 어렵다.
    가급적 Setter를 자제하고 변경용 비즈니스 메서드를 사용하는 것이 좋다.
    그러나 Setter를 썼을 때 복잡한 문제가 쉽게 풀린다던가, 연관관계 편의 메서드가 꼭 필요한 경우 등에 한정하여
    제한적으로 Setter를 허용하는 것도 문제를 단순화할 수 있는 좋은 방법이다.

#### 값 타입 Embedded 객체는 Setter를 제공하지 마라.

    값이라는 것은 원래 immutable한 것이다. 생성 시점에 모두 초기화를 끝내고, 변경할 일이 있으면
    새로운 객체로 만들어서 변경해주자.

#### 모든 연관관계는 지연로딩으로 설정해라.

    연관관계가 LAZY가 아니라 EAGER이라면, 필요 없는 데이터를 한번에 불러올 수 있는 문제가 생길 뿐 아니라
    컬렉션같은경우 너무많은 데이터가 한꺼번에 조회될 수 있기 때문에 지양해야 한다.
    반드시 LAZY로 설정하고 fetch join을 하는 방식을 채택하자.

#### 컬렉션은 필드 초기화 해야한다.

    컬렉션을 영속화하면 Hibernate에서 자신들이 관리하기 편하도록 정해진 Class로 한번 감싸버린다.
    만약에 null값이 들어가있으면, 이후에 싸여진 객체 자체가 변경되는 것이기 때문에 의도하지 않은대로
    동작할 수 있다. 그러므로 반드시 필드 초기화를 하고, 또한 주소를 변경하는 행위를 해선 안되겠다.
    
    또한 NPE 문제에서도 안전할 수 있다.

#### cascade는 그 생명주기가 온전히 종속될 때만 사용해라.

    예를 들어 댓글은 글에 완전히 종속되고, 다른 곳에서 연관으로 사용하지 않는다면, 글에서 댓글을 cascade로
    걸어주면 된다. 그러나 다른 곳에서도 참조하거나 생명주기가 온전히 종속되지 않는다면, 심각한
    장애로 이어질 수 있다.

    참고로 글에서 댓글을 cascade를 건다면, 댓글을 따로 영속화해주지 않아도 글이 영속화 되는 시점에
    함께 영속화가 된다.

#### @Column에서 제공하는 속성들을 적극 활용하자.

    유니크 제약조건, updatable, nullable 등의 속성을 명확히 지정해주는 것이 좋다.

#### Embedded 값 타입을 적극 활용하자.

    테이블의 복잡도는 낮추면서 관심사를 명확하게 분리할 수 있는 방법이다.

#### Builder를 생성자에 사용해주자.

    객체 자체에 Builder를 붙이면, 생성의도가 명확하지 않을 수 있다. 그러나 생성자에 붙이면
    생성 의도가 명확하다.

#### Enum TYPE 은 반드시 String을 사용해라.

    String 말고 Ordinal Type으로 사용하게 되면, 숫자로 관리된다.
    이런 경우 새로운 인자가 추가된다던가 하는 경우에 문제로 이어질 수 있다.

#### Id의 이름은 TABLE_ID 로 설정하는 것이 관리 측면에서 명확하고 편하다.

    말 그대로다.

#### 검색시 FULL Scan을 하지 않도록 Index를 적극적으로 활용해라.

    MySQL에서 기본적으로 생성해주는 Index는 PK 와 UNIQUE Column에 대해서이다.
    이 외의 컬럼이 검색조건으로 들어간다면, Index를 달아주어야 한다.
    Index가 있는것과 없는것은 성능차이가 많이난다.

#### MappedSupperClass 상속으로 (생성일, 수정일, 생성자, 수정자) 를 쉽게 관리해라.

    비즈니스적 측면에서 활용도가 매우 높다.


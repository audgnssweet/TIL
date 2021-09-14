## QueryDSL

    JPA에서 지원하는 Query Library로, JPQL을 Java Code로 작성할 수 있게 해주는 것.

    기존의 JPQL은 문자로 작성되었기 때문에, 컴파일 시점이 아니라 런타임 시점에 문제가 발견된다.
    그러나 QueryDSL을 사용한다면 Java Code여서 Compiler가 문법오류를 잡아줄 수 있다.

    또한 복잡한 쿼리를 Java Code로 작성함으로써 부담감이 덜하고,
    
    동적 쿼리 문제도 기존의 JPQL에서는 문자열 더하기로 해결하거나,
    Criteria 를 사용해서 직관적이지 않은 코드를 작성했어야 했는데
    QueryDSL에서는 쉽게 해결하고, 재사용성까지 높일 수 있다.

    문법이 SQL과 비슷해서 배우기 쉽다

    자동완성의 도움을 받을 수 있다.
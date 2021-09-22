## Java ArrayList vs. Vector

    이 둘은 StringBuilder, StringBuffer처럼 동기화 지원 여부로 나뉜다.
    
    ArrayList는 동기화를 지원하지 않기 때문에, 멀티스레드 환경에서 공유자원으로 활용될 시 위험할 수 있다.
    그러나 싱글스레드 환경에서 조금 더 나은 성능을 제공하고,

    Vector는 동기화를 지원한다.
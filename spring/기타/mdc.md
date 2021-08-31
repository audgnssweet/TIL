<h3> MDC </h3>

    MDC란, java의 Map 형식을 이용하여 클라이언트 특징적인 데이터를 저장하기위한 매커니즘이다.

    조작 함수
    get, put, remove 등 key와 value로 값들을 관리한다.

---

<h3> Spring log 에서의 MDC </h3>

    logback, slf4j 와 같은 java의 logging library 에서는 request가 지나는 경로,
    즉 클래스와 메서드의 스택을 트레이싱하며 로깅을 남긴다.

    그런데 알다시피 springboot는 멀티프로세스 환경이기 때문에 spring bean으로 싱글턴으로
    관리되는 클래스들에서 여러 request들이 멀티스레드 환경으로 거쳐가게 되면, context switching을 하며
    로그가 섞일 수 있게 된다.

    이를 해결하고자 ThreadLocal이라는 변수를 사용할 수 있는데,
    이는 한 쓰레드가 시작되어 끝날때까지 유지되는 변수이다.

    다시 원점으로 돌아가서
    java의 logging library들에서는 ThreadLocal을 통한 CorrelationID 뿐 아니라 map 형식으로 여러 메타데이터들을 삽입할 수 있도록
    MDC를 제공한다.

    그래서 요청에 대한 다양한 메타데이터를 log의 MDC에 넣어주면 의미있는 로그를 출력할 수 있다.

<h3> 왜 공부하게 되었는가? </h3>

    최근에 Server Error Bot을 개발하는 과정에서, Logback의 MDC라는 것을 이용하여 map형식으로
    request에 대한 정보를 저장하고 이를 꺼내어 logging할 때 활용하는 경험을 했다.

    그래서 MDC가 뭔지 궁금해져서 공부해보았다.
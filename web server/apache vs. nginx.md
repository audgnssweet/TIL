<h3> apache vs. nginx </h3>

    둘다 web server라는 공통점이 있다. 

    1. apache
    
    apache는 process 기반을 전제로 한다.
    그 중에서도 MPM-preworker (multi-process-module) 방식은 요청시마다 새로운 process를 할당하는 방식이다.
    MPM-worker 방식은 preworker방식처럼 요청마다 프로세스를 생성하는 방식은 아니지만,
    프로세스와 여러 스레드를 병행해서 사용하는 방식이다.

    MPM 방식의 문제점은 접속자가 많아지면 프로세스를 너무 많이 사용하게 되어
    메모리가 많이 필요하고, 그에 따라 부하가 걸릴 수 있게 된다는 것이다.

    2. nginx

    nginx는 이러한 apache의 MPM방식의 문제점을 다른 방식으로 해결하고 있다.
    바로 event-driven 처리방식을 사용한다는 것이다. 
    
    싱글 프로세스인 master process가 존재하고, IO 이벤트를 감지하는 reactor가 존재한다.
    이를 적절한 worker process로 전달하기 위한 handler도 중간에 위치하고 있다.

    한 개 또는 고정된 개수의 프로세스만 생성하고 해당 프로세스 내부에서 비동기 방식으로 작업을 처리한다.
    작업중 IO, socket read/write 등 CPU가 관여하지 않는 작업이 시작되면 처리가 끝날때까지 동기로 기다리는 것이 아니라
    바로 비동기로 다른 작업을 수행하고, IO가 끝나면 다시 작업을 재개한다.
    
    그러나 긴 IO 처리가 필요한 작업의 경우 (예상으론 동영상 스트리밍 등?)에는 큐에 요청이 쌓여 성능저하 우려가 있다.

    또한 reverse proxy server로 활용하여 was의 부하를 줄일 수 있는 로드밸런서로 활용하기도 한다.

---

nginx

![image](https://user-images.githubusercontent.com/19279163/130990993-0e47f778-9282-4982-9953-db689282c422.png)
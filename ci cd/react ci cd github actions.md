<h3> CI CD With Github Actions & AWS Code deploy </h3>

    Github Action으로 React CI CD 설계하기
    - 삽질 너무 오래해서 내가보려고 만듦

---

    AWS세팅, S3에 업로드하는 과정까지는 같다.

---

    당연하게도 spring boot는 build시 .jar파일이 되므로, JVM (openjdk)만 설치해주면 되는데,
    react는 build시 static한 web resource로 변환이 되는 것을 확인했다.
    
    또한 react를 nginx를 이용해서 deploy해야하는 이유는 크게 2가지가 있다.
    nginx가 event-driven 방식으로 transaction을 처리하기 때문에, 동시접속자가 많아도 적절히 처리가 가능하다.
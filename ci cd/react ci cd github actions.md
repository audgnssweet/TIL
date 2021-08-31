<h3> CI CD With Github Actions & AWS Code deploy </h3>

    Github Action으로 React CI CD 설계하기
    - 삽질 너무 오래해서 내가보려고 만듦

---

    AWS세팅, S3에 업로드하는 과정까지는 같다.

    spring boot는 build시 .jar파일이 되므로, JVM (openjdk)만 설치해주면 되는데,
    react는 build시 static한 web resource로 변환이 되는 것을 확인했다.

    또한 react를 nginx를 이용해서 deploy해야하는 이유는 크게 2가지가 있다.
    nginx가 event-driven 방식으로 서버자원을 효율적으로 활용하기 때문이고 
    react production 파일이 static한 파일이기 때문에, 클라이언트 랜더링으로는 sub-url로 바로 접속이 안돼서
    서버 렌더링이 필요하기 때문이다.

---

    EC2 세팅하기 (권한설정 등의 세팅과정은 생략하겠다)

    Ruby와 AWS Codedeploy까지는 설치된 시점부터 시작한다.

    1. Nginx를 통해 배포할 것이므로 Nginx를 설치해준다.

    sudo apt-get install nginx -y

    (nginx 삭제는) sudo apt-get purge -y nginx*

    2. Nginx 설정파일을 변경해준다.
    
    ubuntu Linux기준, nginx의 설정파일은 
    /etc/nginx/sites-available/default
    이다. 관리자 권한이 필요하니 sudo vim으로 default파일을 수정해주자.

    sudo vim을 통해 파일에 접근하면, location을 설정해주는 부분이 있다.

    그 중에 root = ********* 이런식으로 static한 web자원의 root폴더가 어디인지를 설정해주는 부분이 있는데,
    이 부분을 우리가 build한 react프로젝트의 build경로에 지정해주면 된다.

    실제로 build한 react프로젝트의 루트경로에는 index.html이 있고,
    설정파일에 보면 해당 경로에서 어떤 파일을 메인으로 불러올 것인지를 설정할 수 있다.
    -> 디폴트로 index.html이 설정되어 있다. 건드릴 필요 없다.

    즉, 변경된 설정에서는 nginx의 기본포트인 80번 포트로 접근시 
    react app이 build된 build폴더 안의 (java로 치면 main이라 할 수 있는) index.html을 제공하는 것이다.

    또 하나 중요한 것은
    location / {
        try_files $uri /index.html;
    }
    위처럼 try_files 옵션에 /index.html 로 react app으로 돌려주어야 한다는 것이다.

    3. 설정파일이 잘 적용될 수 있는지 확인한다

    sudo nginx -t
    위 커맨드를 통해 설정파일에 문제가 없는지 테스트해볼 수 있다. 결과는 shell에 찍힌다.
    success라고 뜨면, 설정파일에 문제가 없고 잘 실행된다는 뜻이다.

    4. application-start 시점에 실행할 sh 커맨드파일을
    sudo service nginx restart로 재시작하도록 설정해주면 자동으로 CD가 되어있음을 확인할 수 있다.

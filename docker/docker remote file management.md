<h5> Docker 원격 파일 버전관리 </h5>

    파일을 변경하기 위해 매번 도커 컨테이너에 직접 접속하여 파일을 건드리는 일은,
    번거로울 뿐 아니라 리스크도 존재한다. 

    그래서 Docker에서는 Docker안의 Directory를 Host의 Directory와 연결시킬 수 있는 기능을 제공한다.

    Docker run시 -v (volume) 옵션을 주면 된다.
    -> Docker Docs에서는 [Bind mount a volume] 이라고 설명하고 있다.

---

### apache에 연동하기

    docker run -p 80:80 -v C:\Users\audgn\Downloads\htdocs/:/usr/local/apache2/htdocs/ myws

---

### Nginx에 연동하기

    docker hub에 official image에 잘 설명이 되어있다.

    docker run --name mynginx -p 80:80 -d -v C:\Users\audgn\dockerexam:/usr/share/nginx/html nginx
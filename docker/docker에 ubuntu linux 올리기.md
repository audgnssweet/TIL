<h5> Docker에 ubuntu linux 올리기 </h5>

    docker에 ubuntu linux를 올리려니
    기존의 web server등을 올릴 때와는 달리
    run에 별다른 옵션을 주지 않으니 바로 container가 멈춰버리는 현상이 있었다.

---

    ubuntu는 운영체제여서 그런지, run시에도 -it 옵션을 주어
    1. container로 stdin을 돌린다.
    2. 명령어 기반의 텍스트 입출력 인터페이스 (tty)를 제공한다.

---

정리

    docker run -it -d --name myubuntu ubuntu
    와 같이 올려주고

    docker exec -it myubuntu bash 
    로 접속해준다.
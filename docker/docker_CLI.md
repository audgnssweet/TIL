<h2> 자주 사용하는 Docker CLI </h2>

---

    pull : docker pull [OPTIONS] NAME[:TAG|@DIGEST]

    image : docker images [OPTIONS] [REPOSITORY[:TAG]]

    run : docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
    -> COMMAND는 Container안에서 실행하고싶은 명령어를 미리 지정할 수 있다.

    ps : docker ps [OPTIONS]
    -> -a (all) 옵션

    remove image : docker rmi [OPTIONS] IMAGE [IMAGE...]

    log : docker logs [OPTIONS] CONTAINER
    -> -f 옵션

    remove container : docker rm [OPTIONS] CONTAINER [CONTAINER...]
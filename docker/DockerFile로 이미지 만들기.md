## Dockerfile로 이미지 만들기

    docker에서 image를 만드는 방법은 두 가지가 있다.
    첫 번째는 commit, 두 번째는 dockerfile을 이용한 build 이다.

    commit은 현재 실행중인 container에서 image를 추출해내는 개념이기 때문에
    사실상 백업의 역할이 더 크다.

    dockerfile 을 통한 build는 dockerfile에 적어놓은 내용을 통해
    완전히 새로운 image를 만들어내는 것이기에, 새로운 생성의 역할이다.


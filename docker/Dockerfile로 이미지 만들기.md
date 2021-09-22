## Dockerfile로 이미지 만들기

    docker에서 image를 만드는 방법은 두 가지가 있다.
    첫 번째는 commit, 두 번째는 dockerfile을 이용한 build 이다.

    commit은 현재 실행중인 container에서 image를 추출해내는 개념이기 때문에
    사실상 백업의 역할이 더 크다.

    dockerfile 을 통한 build는 dockerfile에 적어놓은 내용을 통해
    완전히 새로운 image를 만들어내는 것이기에, 새로운 생성의 역할이다.

---

### 기본예제

    nginx:latest 이미지로부터 Dockerfile을 통해 나만의 nginx 이미지 만들기

    원하는 디렉토리에 Dockerfile을 만들어준다.

```dockerfile
FROM nginx:latest
```

    위가 Dockerfile의 내용이다.
    별거 없지만, FROM 이 '특정 이미지로부터' 라는 의미이다.

    그리고 해당 디렉터리로 이동한다.

```shell
docker build -t mydockerfilenginx .

docker build -t [repository이름]:[tag이름] [디렉토리 경로]
```

    현재 디렉토리에서 Dockerfile을 찾아 내용대로 build하여, 이미지를 만들되
    repository의 이름은 mydockerfilenginx, tag는 설정하지 않으므로 latest로 하겠다.

![image](https://user-images.githubusercontent.com/19279163/134316129-bd26a5e6-ebfd-4891-885c-5c4ee35877ea.png)

    위처럼 이미지가 잘 생성된 것을 알 수 있다.

## Dockerfile 작성법

```dockerfile
FROM [repository]:[tag]
RUN apt update && apt install -y nginx
WORKDIR /usr/share/nginx/html
RUN echo "<h1>Hello Docker!</h1>" > index.html
COPY ["index.html", "."]
CMD ["python3", "-u", "-m", "http.server"]
```

    FROM [repository]:[tag]
    이 이미지로부터 시작하겠다. 없다면 dockerhub에서 pull해서라도

    RUN apt update && apt install -y nginx
    shell 명령어 실행 (&&는 앞의 명령어가 성공하면 뒤의 명령어 실행. 반드시 -y 옵션 붙여줘야)

    WORKDIR /usr/share/nginx/html
    디렉토리가 없다면 만들고 이동, 있다면 바로 이동 (앞으로의 명령어는 이 디렉토리에서 실행)

    RUN echo "<h1>Hello Docker!</h1>" > index.html
    해당 디렉토리 안에 index.html에 Hello Docker! 를 넣겠다.

    COPY ["index.html", "."]
    호스트의 현재 디렉토리의 index.html을 현재 WORKDIR에 복사하겠다.

    CMD ["python3", "-u", "-m", "http.server"]
    docker image가 올라간 container에서 해당 명령어를 실행하겠다.

### RUN과 CMD 차이
    
    둘 다 명령어를 실행하는 것은 같으나,
    RUN은 image를 build할 때 반영되는 명령어기 때문에 이미지 자체에 반영되고
    CMD는 image를 run 한 후에 반영되는 명령어기 때문에 container에만 반영된다.

    또한 CMD는 자동으로 실행되지 않도록 run 시에 마지막에 명령어를 지정해줄 수 있다.
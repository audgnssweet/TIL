🎯 NGINX rate limit 으로 반복요청 방어하기
=
    프로젝트를 진행하는데 특정 요청을 반복해서 보내는 행위를 막아달라는 요청이 있었다.
    이를 달성하기 위해서는 rate limiter를 활용해야 했다.

    rate limiter 는 종류가 여러개인데, 대표적으론 guava의 rate limiter 등이 있다고 한다.
    (다음에 알아보자..)

    결론부터 이야기하면 rate limit 을 할 수 있는 방법은 여러가지가 있겠지만 대표적으로
    1. 어플리케이션단 (user 기반)
    2. nginx단 (IP 기반)
    이 있다고 한다.

    내 생각에는 결국 완벽하게...? 방어를 하기 위해서는 두 군데 모두에서 제제가 이뤄져야 한다고 생각한다.
    왜냐하면 nginx 단에서는 IP 기반으로 제어를 하기 때문에 특정 IP 대역에서 사용자가 몰릴 때 문제가 생길 수 있기 때문이다 (나중에 트래픽이 많아질 시 결국에는 해야함)

## 📌 적용하기

    nginx 의 설정파일을 건드려준다.
    nginx.conf, sites-available 의 설정파일 뭐든 상관 없다.

1. nginx.conf 파일

```vi
http {
    limit_req_zone $binary_remote_addr zone=limit:10m rate=30r/s;

    server {
        location / {
            limit_req zone=limit burst=20 nodelay;
        }
    }    
}
```

2. sites-available 의 설정파일

```vi
limit_req_zone $binary_remote_addr zone=limit:10m rate=30r/s;

server {
    location / {
        limit_req zone=limit burst=20 nodelay;
        limit_req_status 429;
        limit_req_log_level error;
    }
}
```

<br>

## 📜 분석

1. limit_req_zone

        요청을 제어할 새로운 그룹을 정의한다.

2. $binary_remote_addr

        nginx default 변수로, request 를 보낸 IP 를 의미한다.

3. zone=limit

        요청을 제어할 그룹의 별칭이다. (아무거나 붙이면 된다)

4. :10m

        zone 에서 활용 가능한 shared memory size 이다.

5. rate=30r/s

        request 제한을 설정하는 것으로, 
        request per second (r/s) 혹은
        request per minute (r/m) 으로 써준다.

        1r/s 면 (해당 IP 에서는 초당 1번 제한)
        1m/s 면 (해당 IP 에서는 분당 1번 제한)
        10m/s 면 (해당 IP 에서는 분당 10번, 즉 6초당 1번)

        으로 제한한다.

6. limit_req zone=limit

        실제로 제한을 하기위해 설정해주는 부분이며, 위에서 정의한 그룹의 이름을 넣어주면된다.

7. burst=20

        제한을 넘어서는 응답이 오는 경우 큐에 저장할 것이며, 그 큐의 사이즈는 20 request로 하겠다는
        의미이다.

8. nodelay

        큐에서 기다리지도 못할 정도로 온 요청들은 기다리지 않고 바로 에러 응답을 하겠다는 의미이다.

9. limit_req_status 429

        request 제한에 걸린 요청들은 기본적으로 503 service unavailable 응답을 받게 되어있다.
        하지만 이를 수정해줄 수 있다.
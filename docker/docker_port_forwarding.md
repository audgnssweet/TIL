<h5> Docker port forwarding </h5>

    Docker에서 Port forwarding이 필요한 이유?

![image](https://user-images.githubusercontent.com/19279163/130306684-cf2daec3-7816-4749-9e98-80644f11e0e7.png)

    Host내의 각 Container 위에서는 독립적인 Port를 갖기 때문이다.
    어플리케이션에서 직접 지정하거나 혹은 디폴트로 가지는 포트를
    실제 호스트의 포트와 연결시켜주지 않으면 네트워크를 통해 접속하는것이 불가능하다.

    port forwarding을 하려면
    Docker container를 올릴시 -p[host-port]:[container-port]처럼 option을 주면 된다.
<h3> Springboot HTTPS 설정하기 </h3>

    요즘은 네트워크상에서 REST API를 사용한다면 HTTPS 설정은 기본이다.
    
    Spring boot에서 HTTPS 설정을 하는 방법을 알아보자.

    이 글은 Let's encrypt의 무료 HTTPS 기준이다.

---

    준비물

    1. AWS 인스턴스
    2. OpenJDK
    3. Nginx

---

    Spring boot 자체에 HTTPS 설정을 하는 것보다는 Nginx를 통해
    Reverse Proxy를 설정해주는 것이 좋다.
    
    가격 면에서도 가장 저렴하고, 부트에서 직접 HTTPS 설정을 해주지 않아도
    Nginx에 HTTPS가 설정되어 있다면 Reverse Proxy를 통해 HTTPS를 누릴 수 있다.

    대충 아키택쳐는 아래 그림과 같다.

![image](https://user-images.githubusercontent.com/19279163/131491254-3a951955-e2b2-427c-aa5b-9449622610a0.png)

    이번 설명에서는 Springboot 한대 기준으로 설명하겠다.

    Reverse proxy를 통해 아래 그림처럼 Nginx가 Spring boot로 request를 전달한다.

---

시작!

    아래 과정은 Nginx까지 설치되어 있다는 가정 하에 진행한다.
    
    1. nginx 설정을 변경해준다.

    nginx 설정파일은 /etc/nginx/sites-available/default 이다.
    sites-available 안의 설정파일이 link되어 sites-enabled로 들어간다.
    커스텀 설정파일을 만들고 싶다면 sites-available에 만들어서 link를 걸어주면 된다.

    해당 설정파일에서 설정해줘야 하는 부분은 딱 두가지이다.

        a. server_name _; 라고 되어있는 부분에 _ 에 자신의 도메인을 입력해준다.

        b. nginx로 reverse proxy를 해줄 것이기 때문에, location / {} 대괄호 안을 지우고
        아래 내용으로 채운다.
        
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection keep-alive;
                proxy_cache_bypass $http_upgrade;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;

        위 내용은 들어오는 설정을 spring boot가 돌아가는 포트로 넘기는 작업이다.

    2. Certbot을 설치한다.
    Certbot은 Let's encrypt 인증서를 자동으로 발급 및 갱신해주는 프로그램이다.
    
    sudo apt-get install -y python-certbot-nginx

    경험상, 이 프로그램은 설치된 nginx에 프로그램을 깔아주는 것이기 때문에 
    중간에 만약 Nginx를 지웠다가 재설치하게된다면, Certbot도 재설치 해줘야한다.

    3. Let's Encrypt를 발급해준다.

    sudo certbot --nginx -d api.example.com

    이 부분에서 해당 EC2가 담당하는 도메인이 두개 이상이라면
    -d 옵션을 더 붙여주면서 뒤에 나열하면 된다.

    발급 중간에, 여러 옵션들이 나오는데 http 요청으로 들어와도 nginx에서 알아서 https로
    돌려줄지 물어보는 부분이 있다. 거의 모든 설정을 2번을 눌러서 마무리해주면 된다.

    그럼 certbot이 알아서 nginx의 설정을 만져 아래와 같은 설정으로 만들어 줄 것이다.

    4. nginx를 재시작해준다.

    sudo service nginx restart

    https가 적용된 모습을 볼 수 있을 것이다.

---

    server {
        root /var/www/html;

        index index.html index.htm index.nginx-debian.html;

        server_name api.example.com;

        location / {
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection keep-alive;
                proxy_cache_bypass $http_upgrade;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
        }


    listen [::]:443 ssl ipv6only=on;
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/api.example.com-0001/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com-0001/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    }

    server {
        if ($host = api.example.com) {
            return 301 https://$host$request_uri;
        }

        listen 80 default_server;
        listen [::]:80 default_server;

        server_name api.example.com;
        return 404; 
    }

---
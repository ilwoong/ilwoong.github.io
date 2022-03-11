---
layout: post
title: docker 컨테이너로 구동되는 nginx에서 reverse proxy 설정하기
tags: [nginx, reverse proxy, docker]
author: ilwoong
excerpt_separator: <!--more-->
---

Home Assistant를 구동하다 보면 포트가 다른 여러 서버를 구동해야 할 일이 생깁니다. 처음 설치할 때는 포트번호를 잘 기억하는데, 설치한지 며칠이 지나면 포트번호를 기억 못해 원하는 서버에 접속할 수 없게 됩니다. 이 문제를 해결해보고자 nginx를 이용하여 reverse proxy를 구축해보기로 하였습니다.

<!--more-->

### 1. nginx 설치하기

nginx를 설치하기 위한 docker-compose.yml 파일은 다음과 같습니다. 추후에 duckdns를 이용하여 인증서를 설치하기 위해 letsencrypt 관련 폴더도 함께 추가해줍니다.

```yaml
version: '3'
services:
  nginx:
    container_name: nginx
    image: nginx:stable
    restart: always
    volumes:
      - /data/nginx/templates:/etc/nginx/templates
      - /data/nginx/ssl:/etc/nginx/ssl
      - /etc/letsencrypt:/etc/letsencrypt
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "80:80"
      - "443:443"
    privileged: true
    environment:
      - TZ=Asia/Seoul
```

### 2. nginx.conf 파일 작성하기

/data/nginx/nginx.conf 파일을 아래와 같이 생성합니다.

```bash
sudo mkdir -p /data/nginx
sudo nano /data/nginx/nginx.conf
```

아래처럼 내용을 작성합니다.

```conf
user nginx;
worker_process auto;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    proxy_intercept_erros on;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Server $host;
    proxy_set_header X-Real-IP $remote_addr;

    include       /etc/nginx/mime.types;
    default-type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    include /etc/nginx/conf.d/*.conf;
}
```


### 3. default.conf.template 파일 작성하기

nginx에서 기본적으로 로드하는 파일은 /etc/nginx/conf.d/default.conf 파일입니다. 이 파일은 기본으로 생성되며, 다음과 같은 내용을 담고 있습니다. 이 파일의 설정에 의해 80포트를 사용하게 되므로, 이 파일을 덮어써야 합니다.

```conf
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

nginx는 templates 폴더 밑의 xxxx.conf.template 파일을 conf.d 폴더의 xxxx.conf 파일로 자동 변환합니다. 위의 default 내용을 변경하기 위해서 아래처럼 default.conf.template 파일을 생성합니다.

```bash
sudo mkdir -p /data/nginx/templates
sudo nano /data/nginx/templates/default.conf.template
```

테스트 페이지를 하나 만들기 위해 python의 웹 서버 기능을 사용해봅니다. 아래 명령은 현재 폴더 위치를 웹 서버로 오픈해주는 명령입니다. port 번호는 생략할 수 있으며, 생략하면 8080을 사용합니다.

```bash
python3 -m http.server {port}
```

필요없는 주석을 제거하고 reverse proxy 부분을 추가하면 다음과 같습니다. 

```conf
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # 프록시로 연결할 주소 추가
    # http://서버주소/proxy_test 로 접속 가능
    location = /proxy_test {
        proxy_pass http://192.168.1.2:8080;
    }

    # redirect
    location = /redirect_test {
        return 301 http://192.168.1.2:8080;
    }
}
```

location을 정의할 때 다음과 같은 문법을 사용할 수 있습니다.

```
# 정확하게 일치
# ex) http://192.168.1.2/exact_dst
location = /exact_dst {
    ...
}

# 지정한 패턴으로 시작
# ex) http://192.168.1.2/dst123
location /dst {
    ...
}

# 지정한 패턴으로 시작
# ex) http://192.168.1.2/documents/readme.html
location /documents/ {
    ...
}

# 지정한 패턴으로 시작 패턴이 일치 하면 다른 패턴 탐색 중지
# ex) http://192.168.1.2/images/
location ^~ /images/ {
    ...
}

# 정규식 표현 일치 - 대소문자 구분
# ex) http://192.168.1.2/image.jpg
location ~ \.(gif|jpg|jpeg)$ {
    ...
}

# 정규식 표현 일치 - 대소문자 구분 안함
# ex) http://192.168.1.2/image.JPG
location ~* \.(gif|jpg|jpeg)$ {
    ...
}
```

모든 설정이 끝나면 nginx를 재시작해줍니다.

```bash
docker-compose restart nginx
```

### 참고자료
- https://lahuman.github.io/nginx_location_options/
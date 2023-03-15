---
layout: post
title: Home Assistant 설치하기 (with Docker, NGINX)
tags: [Raspberry Pi, Home Assistant, Docker, NGINX]
author: ilwoong
excerpt_separator: <!--more-->
---

예전에 사용하다 서비스를 해지한 U+ IoT 플러그가 집에 2개 남아있었는데, U+ 서비스 가입하지 않고 이 플러그들을 사용하기 위해서는 Home Assistant 서버가 하나 필요하다는 것을 확인하게 되었습니다. 집에 남아있던 라즈베리파이에 Home Assistant를 설치해서 사용하기로 했으며, 추후에 다시 설치할 일이 있을 때를 대비하여 설치 과정을 기록으로 남깁니다. **2021년 4월 30일에 작성했던 글을 최근 작업한 내용으로 수정합니다.**

<!--more-->

### 라즈베리파이 OS 설치

혹시 나중에 다른 서비스들을 올리고 싶을 수도 있어서, Raspberry Pi OS를 설치하였습니다. 서버로만 동작시킬 것이고, 가능하면 용량 및 소비전력을 최소화하기 위해서 Raspberry Pi OS **Lite 버전**을 설치하였습니다. 설치 과정은 여러 곳에 잘 나와있으니 생략합니다.

### Docker 설치

Raspberry Pi OS에 바로 Home Assistant를 설치할 수도 있겠지만, 추후 업그레이드나, 깔끔하게 삭제해야 할 필요가 있을 때를 대비하여 도커 컨테이너로 설치합니다. 우선 도커를 다음과 같이 설치합니다.

```bash
sudo apt update
sudo apt install docker.io docker-compose
```

WSL에서 설치할때와 마찬가지로 실행할 때마다 권한 문제가 생기는 것이 귀찮기 때문에 현재 user를 docker 그룹에 넣어줍니다.

```bash
sudo usermod -aG docker pi
```

서비스를 실행시키고 로그아웃을 한번 했다가 다시 로그인하면 docker를 사용할 수 있습니다.

```bash
sudo service docker start
logout
```

### docker-compose.yml 작성

docker-compose를 이용하기 위해 다음과 같이 docker-compose.yml 파일을 생성합니다. ```/data``` 폴더는 docker container들의 작업 파일들을 저장하기 위한 폴더로 미리 만들어뒀습니다. 제가 사용하는 서비스는 총 4가지입니다.

- [NGINX](https://www.nginx.com/){:target="_blank"}
- [Home Assistant](https://www.home-assistant.io/){:target="_blank"}
- [Mosquitto MQTT broker](https://mosquitto.org/){:target="_blank"}
- [Zwavejs2mqtt](https://zwave-js.github.io/zwavejs2mqtt/){:target="_blank"}

```yml
version: '3'
services:
  nginx:
    container_name: nginx
    image: arm64v8/nginx:stable
    restart: always
    volumes:
      - /data/nginx/templates:/etc/nginx/templates
      - /data/nginx/ssl:/etc/nginx/ssl
      - /data/nginx/conf.d:/etc/nginx/conf.d
      - /etc/letsencrypt:/etc/letsencrypt
      - /usr/share/zoneinfo/Asia/Seoul:/etc/localtime:ro
    ports:
      - "80:80"
      - "443:443"
    privileged: true
    environment:
      - TZ=Asia/Seoul

  homeassistant:
    container_name: homeassistant
    image: ghcr.io/home-assistant/raspberrypi4-homeassistant:stable
    volumes:
      - /data/ha:/config
      - /etc/localtime:/etc/localtime:ro
      - /var/log/fail2ban.log:/var/log/fail2ban.log
    ports:
      - "8123:8123"
    restart: unless-stopped
    network_mode: host
    privileged: true
    environment:
      - TZ=Asia/Seoul

  mosquitto:
    container_name: mqtt
    image: eclipse-mosquitto
    volumes:
      - /data/mosquitto/config:/mosquitto/config
      - /data/mosquitto/data:/mosquitto/data
      - /data/mosquitto/log:/mosquitto/log
    restart: always
    network_mode: host

  zwavejs2mqtt:
    container_name: zwavejs2mqtt
    image: zwavejs/zwavejs2mqtt:latest
    restart: always
    tty: true
    stop_signal: SIGINT
    environment:
      - SESSION_SECRET=mysupersecretkey
      - TZ=Asia/Seoul
    networks:
      - zwave
    devices:
      - '/dev/ttyACM0:/dev/ttyACM0'
    volumes:
      - /data/zwavejs/store:/usr/src/app/store
    ports:
      - '8091:8091' # port for web interface
      - '3000:3000' # port for zwave-js websocket server

networks:
  zwave:
```

### NGINX 템플릿 작성 (default)

docker로 설치했기 때문에 docker 이미지 내에 있는 ```default.conf``` 파일이 적용됩니다. 편의를 위해 기본 설정을 변경할 일이 생기기 때문에 아래와 같이 ```/data/nginx/templates``` 폴더에 ```default.conf.template``` 파일을 생성해줍니다. 라즈베리파이에서 동작하고 있는 각 서비스의 포트 번호를 외우고 있기는 힘들기 때문에 redirection을 적용해줬습니다.

```nginx
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;
    server_tokens off;

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

    # homeassistant
    location = /ha {
        return 301 http://192.168.1.2:8123;
    }

    # zwavejs2mqtt
    location = /zwave {
        return 301 http://192.168.1.2:8091;
    }
}
```

### dhparam.pem 파일 생성하기

보안을 위해서 4096 비트 길이의 키를 생성하는게 좋습니다. 라즈베리파이에서 4096 비트 길이의 키를 생성하는데 정확하진 않지만 6~8시간 정도 걸렸던 것 같습니다. 자기 전에 실행시켜두고 일어나서 확인하는 것을 추천합니다. 참고로 2048 비트 길이의 키는 몇 분 안에 만들어집니다.

```bash
sudo openssl dhparam -out /data/nginx/ssl/dhparam.pem 4096
```


### 인증서 설치

[certbot을 이용하여 let's encrypt 인증서 설치하기](/2022/06/01/certbot-letsencrypt){:target="_blank"}를 참고하여 인증서를 설치합니다.


### NGINX 템플릿 작성 (Home Assistant)

DDNS를 사용하기 위해 [duckdns.org](https://duckdns.org)에서 도메인을 생성하고, ```/data/nginx/templates``` 폴더 밑에 ```ha.conf.template``` 파일을 아래와 같이 생성합니다. **```{domain-name}``` 부분은 실제 생성한 도메인 이름으로 변경합니다.**

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

server {
    listen 80;
    server_name {domain-name}.duckdns.org;
    location / {
        return 301 https://{domain-name}.duckdns.org;
    }
}

server {
    # Update this line to be your domain
    server_name {domain-name}.duckdns.org;
    server_tokens off;

    # Ensure these lines point to your SSL certificate and key
    ssl_certificate /etc/letsencrypt/live/{domain-name}.duckdns.org/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/{domain-name}.duckdns.org/privkey.pem;

    # Ensure this line points to your dhparams file
    ssl_dhparam /etc/nginx/ssl/dhparams.pem;

    # These shouldn't need to be changed
    listen [::]:443 ssl  default_server ipv6only=off; # if your nginx version is >= 1.9.5 you can also add the "http2" flag here
    add_header Strict-Transport-Security "max-age=31536000; includeSubdomains";
    # ssl on; # Uncomment if you are using nginx < 1.15.0
    ssl_protocols TLSv1.2;
    ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH:!aNULL:!eNULL:!EXPORT:!DES:!MD5:!PSK:!RC4";
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;

    location / {
        proxy_pass http://192.168.1.2:8123;
        proxy_set_header Host $host;
        proxy_redirect http:// https://;
        proxy_http_version 1.1;
        proxy_buffering off;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
    }

    autoindex_localtime on;
    autoindex on;
}
```

### docker container 실행

docker-compose.yml 파일이 위치한 폴더에서 다음과 같이 실행하면 도커 컨테이너를 실행할 수 있습니다.

```bash
docker-compose up -d
```

Home Assistant 컨테이너가 실행되고 있는 동안에는 <http://localhost:8123>{:target="_blank"} 으로 Home Assistant의 웹 인터페이스에 접속할 수 있습니다. 원격에서 작업중이라면 localhost 대신 라즈베리파이의 ip 주소를 적어주면 됩니다. 참고로 이 포스트에서는 ```192.168.1.2```를 라즈베리파이의 ip 주소로 사용합니다.

### 참고자료

<https://www.home-assistant.io/installation/raspberrypi>{:target="_blank"}
---
layout: post
title: Docker로 Home Assistant 구동하기
tags: [Raspberry Pi, Home Assistant, Docker]
author: ilwoong
---

예전에 사용하다 서비스를 해지한 U+ IoT 플러그가 집에 2개 남아있었는데, U+ 서비스 가입하지 않고 이 플러그들을 사용하기 위해서는 Home Assistant 서버가 하나 필요하다는 것을 확인하게 되었습니다. 집에 남아있던 라즈베리파이에 Home Assistant를 설치해서 사용하기로 했으며, 추후에 다시 설치할 일이 있을 때를 대비하여 설치 과정을 기록으로 남깁니다.

### 라즈베리파이 OS 설치

혹시 나중에 다른 서비스들을 올리고 싶을 수도 있어서, Raspberry Pi OS를 설치하였습니다. 설치 과정은 여러 곳에 잘 나와있으니 생략합니다.

### Docker 설치

Raspberry Pi OS에 바로 Home Assistant를 설치할 수도 있겠지만, 추후 업그레이드나, 깔끔하게 삭제해야 할 필요가 있을 때를 대비하여 도커 컨테이너로 설치합니다. 우선 도커를 다음과 같이 설치합니다.

```bash
$ sudo apt install docker.io docker-compose
```

WSL에서 설치할때와 마찬가지로 실행할 때마다 권한 문제가 생기는 것이 귀찮기 때문에 현재 user를 docker 그룹에 넣어줍니다.

```bash
$ sudo usermod -aG docker pi
```

### Home Assistant 설치하기

docker 명령어를 이용하면 다음과 같이 Home Assistant를 설치할 수 있습니다. PATH_TO_YOUR_CONFIG는 설정파일을 저장할 경로입니다. 저는 /data/homeassistant로 설정해보겠습니다.

```bash
$ docker run --init -d \
  --name homeassistant \
  --restart=unless-stopped \
  -v /PATH_TO_YOUR_CONFIG:/config \
  -v /etc/localtime:/etc/localtime:ro \
  --network=host \
  homeassistant/raspberrypi4-homeassistant:stable
```

위 명령어를 타이핑하기 귀찮기 때문에 docker-compose 를 이용하기 위해 다음과 같이 docker-compose.yml 파일을 생성합니다.

```yml
version: '3'
services:
  homeassistant:
    container_name: homeassistant
    image: homeassistant/raspberrypi4-homeassistant:stable
    volumes:
      - /data/homeassistant:/config
      - /etc/localtime:/etc/localtime:ro
    restart: unless-stopped
    network_mode: host
```

docker-compose.yml 파일이 위치한 폴더에서 다음과 같이 실행하면 도커 컨테이너를 실행할 수 있습니다.

```bash
$ docker-compose up -d
```

Home Assistant 컨테이너가 실행되고 있는 동안에는 <http://localhost:8123>{:target="_blank"} 으로 Home Assistant의 웹 인터페이스에 접속할 수 있습니다. 원격에서 작업중이라면 localhost 대신 라즈베리파이의 ip 주소를 적어주면 됩니다.

### 참고자료

<https://www.home-assistant.io/installation/raspberrypi>{:target="_blank"}
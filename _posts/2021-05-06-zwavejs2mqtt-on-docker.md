---
layout: post
title: Docker로 zwavejs2mqtt 설치하기
tags: [z-wave, Home Assistant]
author: ilwoong
---

Home Assisant에서 z-wave 기기들을 사용하려면 z-wave 동글이 필요하고, z-wave를 동작시킬 소프트웨어가 필요합니다. 저는 U+ IoT z-wave 동글을 당근마켓에서 중고로 구입하였으며, Home Assistant에서 권장하고 있는 zwavejs2mqtt를 사용하기로 했습니다.

### z-wave 동글 확인하기

usb 포트에 z-wave 동글을 꽂고 나서 제대로 동작하는지 다음과 같이 확인합니다. 제가 사용한 동글은 Aeotec Z-Stick Gen5 (ZW090) 모델임을 알 수 있습니다.

```bash
$ lsusb
Bus 002 Device 002: ID 2109:0813 VIA Labs, Inc. 
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 004: ID 0658:0200 Sigma Designs, Inc. Aeotec Z-Stick Gen5 (ZW090) - UZB
Bus 001 Device 003: ID 2109:2813 VIA Labs, Inc. 
Bus 001 Device 002: ID 2109:3431 VIA Labs, Inc. Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

어떤 시리얼 포트를 사용하고 있는지 다음과 같이 확인합니다.

```bash
$ ls /dev/ttyACM*
/dev/ttyACM0
```

### docker-compose.yml 수정하기

[이전 포스트]({{post_url}}/2021/04/30/ha-on-docker.html)에서 Home Assistant를 docker-compose를 사용해서 설치했습니다. docker-compose.yml 파일을 수정하여 zwavejs2mqtt를 설정해보겠습니다. 

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
  zwave
```

이 중 devices 항목의 {serial_path} 부분을 앞에서 확인한 시리얼 포트 이름을 적어주면 됩니다. 

```yml
    devices:
      - '{serial_path}:/dev/ttyACM0'
```

다음과 같이 도커를 실행시키면 zwave 서버가 실행됩니다.

```bash
$ docker-compose up -d
```

### Home Assistant와 zwavejs2mqtt 연동

zwave-js 웹인터페이스로 접속하기 위해 8091 포트로 접속합니다. 그 이후 설정 > Home Assistant > WS Server 설정 을 켜주면 zwave-js 서버 설정이 완료됩니다.

Home Assistant에 접속하여 <code>설정 > 통합 구성요소 > 통합 구성요소 추가 > Z-Wave JS</code>를 선택하여 Z-Wave 통합 관리요소를 추가하면 Z-Wave 기기들을 Home Assistant에서 관리할 수 있습니다.


### 참고자료
- <https://m.blog.naver.com/PostView.nhn?blogId=fromzip&logNo=221826265696>{:target="_blank"}
- <https://www.home-assistant.io/docs/z-wave/>{:target="_blank"}
- <https://github.com/zwave-js/zwavejs2mqtt/tree/master/docker>{:target="_blank"}
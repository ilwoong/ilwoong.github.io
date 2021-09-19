---
layout: post
title: Home Assistant 오류 - trusted_proxies
tags: [Home Assistant, Trusted Proxies]
author: ilwoong
---

언젠가 Home Assistant 도커 이미지를 업데이트 했더니 접속이 되지 않는 현상이 발생했습니다. 처음에는 nginx나 duckdns 설정이 잘못되었다고 생각했는데 검색해보니 Home Assistant 이미지가 업데이트 되면서 trusted_proxies 부분을 생략할 수 없게 바뀌었기 때문이라고 합니다.

### 에러 파악

Home Assistant의 도커 로그를 확인해보니 아래처럼 시작되지 못하고 자꾸 죽는 것을 확인할 수 있었습니다.

```
homeassistant    | Current thread 0xb6fe2020 (most recent call first):
homeassistant    | <no Python frame>
homeassistant    | [cont-finish.d] executing container finish scripts...
homeassistant    | [cont-finish.d] done.
homeassistant    | [s6-finish] waiting for services.
homeassistant    | [s6-finish] sending all processes the TERM signal.
homeassistant    | [s6-finish] sending all processes the KILL signal and exiting.
homeassistant    | [s6-init] making user provided files available at /var/run/s6/etc...exited 0.
homeassistant    | [s6-init] ensuring user provided files have correct perms...exited 0.
homeassistant    | [fix-attrs.d] applying ownership & permissions fixes...
homeassistant    | [fix-attrs.d] done.
homeassistant    | [cont-init.d] executing container initialization scripts...
homeassistant    | [cont-init.d] done.
homeassistant    | [services.d] starting services
homeassistant    | [services.d] done.
homeassistant    | Fatal Python error: init_interp_main: can't initialize time
homeassistant    | Python runtime state: core initialized
homeassistant    | PermissionError: [Errno 1] Operation not permitted
```

### 해결 방법

Home Assistant의 설정파일인 configuration.yaml 파일에 아래와 같이 추가합니다.

```yaml
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 127.0.0.1
    - ::1
```

위와 같이 하고 Home Assistant 도커를 재시작해도 접속이 되지 않으면, 로그 메시지를 확인해서 접속이 막히는 ip 주소를 trusted_proxies에 추가해줍니다.

혹시 그래도 해걸되지 않으면 아래와 같이 libseccomp2를 최신 버전으로 업데이트 해서 해결했다는 글도 있습니다.

```bash
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 04EE7237B7D453EC 648ACFD622F3D138
echo "deb http://deb.debian.org/debian buster-backports main" | sudo tee -a /etc/apt/sources.list.d/buster-backports.list
sudo apt update
sudo apt install -t buster-backports libseccomp2
```

### 참고

- <https://community.home-assistant.io/t/migration-to-2021-7-fails-fatal-python-error-init-interp-main-cant-initialize-time/320648/8>{:target="_blank"}
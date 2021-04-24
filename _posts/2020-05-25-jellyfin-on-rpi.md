---
layout: post
title: 라즈베리파이에서 jellyfin 설정하기
tags: [Raspberry Pi, jellyfin]
author: ilwoong
---

jellyfin은 오픈소스 미디어서버입니다. 안드로이드/iOS 앱이 존재하고, 브라우저로 접속해서 미디어 파일들을 재생할 수 있습니다. 가족 구성원별로 계정을 생성할 수 있으며, 계정별로 보던 파일의 이어보기 기능 등을 지원합니다.

### jellyfin 설치

jellyfin 저장소를 저장소 리스트에 추가하고, apt를 이용하여 jellyfin을 설치합니다.

```bash
$ sudo apt install apt-transport-https
$ wget -O - https://repo.jellyfin.org/jellyfin_team.gpg.key | sudo apt-key add -
$ echo "deb [arch=$( dpkg --print-architecture )] https://repo.jellyfin.org/$( awk -F'=' '/^ID=/{ print $NF }' /etc/os-release ) $( awk -F'=' '/^VERSION_CODENAME=/{ print $NF }' /etc/os-release ) main" | sudo tee /etc/apt/sources.list.d/jellyfin.list
$ sudo apt update
$ sudo apt install jellyfin
```

### 라즈베리파이 GPU 메모리 할당량 설정

기본 설정으로는 GPU 메모리가 부족할 수 있어 다음과 같이 GPU 메모리 할당량을 변경합니다.

```bash
$ sudo nano /boot/config.txt
```

위 파일의 맨 아랫줄에 다음과 같이 추가합니다.

```conf
gpu_mem = 512
```

위 설정대로 했는데도, 성능이 떨어지거나 하면 라즈베리파이 펌웨어를 최신으로 업데이트 합니다.

```bash
$ sudo rpi-update
```

### 하드웨어 가속 설정

하드웨어 가속 사용을 위해서는 jellyfin FFMpeg 프로세스에서 인코더를 사용하도록 jellyfin 서비스 유저를 비디오 그룹에 추가해야 합니다.

```bash
$ sudo usermod -aG video jellyfin
$ sudo service jellyfin restart
```

그 다음 jellyfin 설정 페이지에 접속해서

```conf
재생 > 하드웨어가속 > OpenMAX OMX 선택 > 하드웨어 인코딩 활성화 선택
```

과 같이 설정하면 트랜스코딩 시 하드웨어를 사용하여 가속하도록 설정할 수 있습니다.


## 참고자료
- <https://jellyfin.org/downloads/>
- <https://blog.dalso.org/raspberry-pi/raspberry-pi-4/7829>
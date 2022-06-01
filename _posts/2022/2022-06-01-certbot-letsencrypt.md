---
layout: post
title: certbot을 이용하여 let's encrypt 인증서 설치하기
tags: [certbot, snapd, letsencrypt]
author: ilwoong
---

WSL에서 Docker를 사용하기 위해서는 Docker Desktop이 설치되어 있어야 했습니다만, Docker Desktop이 유료화됨에 따라 회사에서는 Docker Desktop을 이용해서 WSL에서 Docker를 구동하기 힘들어졌습니다. 이번 포스트에서는 Docker Desktop 없이 WSL에서 genie를 이용하여 Docker를 구동하는 방법에 대해 설명해보겠습니다.

### snapd를 이용하여 certbot 설치하기

먼저 snapd를 설치하고 시스템을 다시 시작합니다.

```bash
sudo apt update
sudo apt install snapd
sudo reboot
```

snapd를 최신 버전으로 업데이트 합니다.

```bash
sudo snap install core
```

certbot을 설치하고, 사용하기 편하도록 /usr/bin 폴더로 symlink를 하나 만들어줍니다.

```bash
sudo snap install certbot --classic
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

### nginx용 인증서 받기

웹서버를 중지시킵니다. 여기서는 docker에서 NGINX로 실행시켰으므로, docker-compose.yml 파일이 있는 폴더로 이동하여 아래 명령어를 실행합니다.

```bash
docker-compose stop nginx
```

기존에 설치된 인증서를 삭제하려면 다음 명령어를 실행합니다.

```bash
sudo certbot delete 
```

인증서를 발급받습니다.

```bash
sudo certbot certonly --standalone -d '{사이트 주소}'
```

웹서버를 다시 실행합니다.

```bash
docker-compose up -d nginx
```

### 참고자료

- https://snapcraft.io/install/certbot/raspbian
- https://happist.com/573990
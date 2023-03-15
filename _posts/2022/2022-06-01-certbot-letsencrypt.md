---
layout: post
title: certbot을 이용하여 let's encrypt 인증서 설치하기
tags: [certbot, snapd, letsencrypt]
author: ilwoong
---

WSL에서 Docker를 사용하기 위해서는 Docker Desktop이 설치되어 있어야 했습니다만, Docker Desktop이 유료화됨에 따라 회사에서는 Docker Desktop을 이용해서 WSL에서 Docker를 구동하기 힘들어졌습니다. 이번 포스트에서는 Docker Desktop 없이 WSL에서 genie를 이용하여 Docker를 구동하는 방법에 대해 설명해보겠습니다.

<br>

### snapd를 이용하여 certbot 설치하기

먼저 snapd를 설치하고 시스템을 다시 시작합니다.

```terminal
sudo apt update
sudo apt install snapd
sudo reboot
```

snapd를 최신 버전으로 업데이트 합니다.

```terminal
sudo snap install core
```

certbot을 설치하고, 사용하기 편하도록 /usr/bin 폴더로 symlink를 하나 만들어줍니다.

```terminal
sudo snap install certbot --classic
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

<br>

### nginx용 인증서 발급받기

웹서버를 중지시킵니다. 여기서는 docker에서 NGINX로 실행시켰으므로, docker-compose.yml 파일이 있는 폴더로 이동하여 아래 명령어를 실행합니다.

```terminal
docker-compose stop nginx
```

기존에 설치된 인증서를 삭제하려면 다음 명령어를 실행합니다.

```terminal
sudo certbot delete
```

인증서를 발급받습니다.

```terminal
sudo certbot certonly --standalone -d '{사이트 주소}'
```

웹서버를 다시 실행합니다.

```terminal
docker-compose up -d nginx
```

<br>

### 인증서 갱신하기

certbot의 --dry-run 옵션을 이용해서 인증서 갱신을 시뮬레이션 해봅니다.

```terminal
sudo certbot renew --dry-run
```

웹서버가 동작하고 있으면 아래와 같은 에러메시지를 출력합니다.

```terminal
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/ilwoong.duckdns.org.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Simulating renewal of an existing certificate for ilwoong.duckdns.org
Failed to renew certificate ilwoong.duckdns.org with error: Could not bind TCP port 80 because it is already in use by another process on this system (such as a web server). Please stop the program in question and then try again.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
All simulated renewals failed. The following certificates could not be renewed:
  /etc/letsencrypt/live/ilwoong.duckdns.org/fullchain.pem (failure)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1 renew failure(s), 0 parse failure(s)
Ask for help or search for solutions at https://community.letsencrypt.org. See the logfile /var/log/letsencrypt/letsencrypt.log or re-run Certbot with -v for more details
```

웹서버를 중단시킵니다.

```terminal
docker-compose stop nginx
```

다시 시뮬레이션을 해 보면 시뮬레이션이 성공하는 것을 확인할 수 있습니다.

```terminal
$ sudo certbot renew --dry-run
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/ilwoong.duckdns.org.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Simulating renewal of an existing certificate for ilwoong.duckdns.org

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations, all simulated renewals succeeded:
  /etc/letsencrypt/live/ilwoong.duckdns.org/fullchain.pem (success)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

```--dry-run``` 옵션을 빼고 ```certbot renew```를 실행하면 인증서가 갱신됩니다.

```terminal
$ sudo certbot renew
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/ilwoong.duckdns.org.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Renewing an existing certificate for ilwoong.duckdns.org

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations, all renewals succeeded:
  /etc/letsencrypt/live/ilwoong.duckdns.org/fullchain.pem (success)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```


### 참고자료

- <https://snapcraft.io/install/certbot/raspbian>{:target="_balnk"}
- <https://happist.com/573990>{:target="_balnk"}
- <https://smoh.tistory.com/406>{:target="_balnk"}
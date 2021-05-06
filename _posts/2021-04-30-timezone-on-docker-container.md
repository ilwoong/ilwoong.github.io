---
layout: post
title: Docker Container에 TimeZone 지정하기
tags: [Docker, TimeZone]
author: ilwoong
---

도커 컨테이너를 이용하여 jekyll을 구동하는 경우, 새 글을 _posts 폴더에 생성하고 jekyll build 를 실행했을 때, 오늘 날짜로 작성한 글들이 미래에서 작성된 글이라며 빌드하지 않는 경우가 발행합니다. 이 경우 jekyll을 구동하는 컨테이너에 TimeZone을 설정해주면 됩니다.

### docker 명령어로 TimeZone 설정하기

아래와 같이 명령어 인자에 TimeZone 관련 변수들을 추가하면 됩니다.

```bash
$ docker run ...
    -v /etc/localtime:/etc/localtime:ro \
    -e TZ=Asia/Seoul \
```

### docker-compose.yml에 TimeZone 설정하기

아래처럼 /etc/localtime을 읽기전용으로 마운트시키고, 환경변수에 TZ를 추가하면 됩니다.

```yml
services:
  myservice:
    ....
    volumes:
      - /etc/localtime:/etc/localtime:ro
    environment:
      TZ: "Asia/Seoul"
```

### 참고자료

- <https://gogorchg.tistory.com/entry/Docker-Container-에-Timezone-설정-하기>{:target="_blank"}
- <https://hakurei.tistory.com/220>{:target="_blank"}
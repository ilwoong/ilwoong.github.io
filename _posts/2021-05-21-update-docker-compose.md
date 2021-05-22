---
layout: post
title: docker-compose로 관리되는 컨테이너의 이미지 업데이트
tags: [docker, docker-compose]
author: ilwoong
---

docker-compose 로 구성한 docker 이미지들의 업데이트는 어떻게 할까 궁금하여 검색해보니 docker-compose.yml 파일이 있는 폴더에서 아래처럼 명령어를 수행하면 됩니다.

### 이미지 pull

```bash
$ docker-compose pull
```

### 컨테이너 생성

```bash
$ docker-compose -up --force-recreate --build -d
```

### 필요없는 이미지 제거

```bash
$ docker image prune -f
```

이 경우 현재 동작하지 않고 있는 모든 이미지가 삭제되므로, 잠시 중지시켜놓은 컨테이너가 없는 경우에만 실행해야 합니다.

### 참고자료
- <https://hakurei.tistory.com/290>{:target="_blank"}
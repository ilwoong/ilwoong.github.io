---
layout: post
title: WSL에서 Docker 구동하기
tags: [WSL, Docker]
author: ilwoong
---

WSL에서 Docker 컨테이너를 실행하기 위해서는 Docker Desktop for Windows가 필요합니다. Docker Desktop을 설치하고, Docker WSL Integration 설정을 완료하면 WSL에서 docker 명령어를 사용하여 Docker 컨테이너를 실행시킬 수 있습니다.

### Docker Desktop for Windows 설치

<https://hub.docker.com/editions/community/docker-ce-desktop-windows/>


### Docker Desktop 에서 WSL2 기능 활성화

Settings > General > Use the WSL2 based engine 체크


### Docker WSL Integration 설정

Settings > Resources > WSL Integration 에서 docker를 실행할 배포판 설정합니다.

### 사용자를 docker 그룹 추가

docker 실행할때마다 나타나는 권한 문제를 해결하기 위해 사용자를 docker 그룹에 추가시킵니다.

```bash
$ sudo usermod -aG docker username
```


### 참고자료
- <https://docs.docker.com/docker-for-windows/wsl/>{:target="_blank"}
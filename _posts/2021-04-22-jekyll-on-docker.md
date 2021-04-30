---
layout: post
title: Docker로 Jekyll 설정하기
tags: [Jekyll, Docker]
author: ilwoong
---

jekyll을 사용하면 마크다운 형식의 문서들로 웹페이지를 쉽게 생성할 수 있습니다. 리눅스에 jekyll을 바로 설치하여 사용할 수도 있으나, 관리의 편의성을 위해 docker 컨테이너로 설치하는 것을 추천합니다.

### ubuntu에 jekyll 설치

jekyll을 설치하는 방법은 다음과 같습니다.

```bash
~ $ sudo apt install ruby ruby-dev
~ $ sudo gem install bundler jekyll
```

새로운 블로그에 사용할 폴더구조를 다음 명령어로 만들어줍니다.

```bash
~ $ jekyll new blogname
```

다음 명령어로 블로그가 동작하도록 실행시킬 수 있으며, 따로 설정을 바꾸지 않았으므로 <http://localhost:4000>으로 접속해볼 수 있습니다.

```bash
~ $ cd blogname
~/blogname $ bundle exec jekyll serve
```

이렇게 설치하면 매우 많은 패키지들이 설치되게 됩니다. 설정을 잘못했거나, 더이상 필요가 없어져서 지우고 싶을때 깔끔하게 지우기가 힘듭니다. 그래서 jekyll을 docker image를 이용해서 설치해보도록 하겠습니다. docker 설치방법은 [이전 포스트]({% post_url 2021-04-21-docker-on-wsl2 %})

### docker에서 jekyll 실행하기

```bash
$ docker run -v="$PWD/blog:/srv/jekyll" -p 4000:4000 -it jekyll/jekyll jekyll serve
```

### docker compose 설치하기

컨테이너를 실행할 때마다 긴 docker 명령어를 타이핑하는 것은 매우 귀찮은 일입니다. docker-compose를 이용하여 좀 더 쉽게 작업할 수 있습니다.

```bash
$ sudo apt install docker-compose
```

### docker-compose.yml 파일 생성하기

```yml
version: '3'

services:
  jekyll:
    container_name: jekyll
    image: jekyll/jekyll
    ports:
      - "4000:4000"
    command: jekyll serve
    volumes:
      - $PWD/jekyll:/srv/jekyll
      - /etc/localtime:/etc/localtime:ro
    environment:
      TZ: "Asia/Seoul"
```

### Docker 컨테이너 실행하기

docker-compose.yml에 적어둔 컨테이너를 실행하기 위해서는 다음 명령어를 실행하면 됩니다.

```bash
$ docker-compose up -d
```

### jekyll theme 적용하기

디자인 감각이 없는 저로써는 웹디자인을 멋지게 할 자신이 없습니다. 그러나 jekyll은 [Jekyll Theme](http://jekyllthemes.org/) 와 같은 사이트에서 테마를 골라 블로그에 적용하여 그럴듯한 웹페이지를 신속하게 만들 수 있습니다. 저는 정리한 내용을 나중에 다시 찾아보는 것이 목적이기 때문에 태그와 검색 기능이 있는 [Type-on-strap](https://github.com/sylhare/Type-on-Strap) 테마를 제 블로그에 적용하였습니다.

### 참고자료

- <https://jekyllrb-ko.github.io/>{:target="_blank"}
- <https://github.com/envygeeks/jekyll-docker>{:target="_blank"}
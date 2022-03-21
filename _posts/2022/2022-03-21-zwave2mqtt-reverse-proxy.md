---
layout: post
title: zwave2mqtt를 reverse proxy 아래에서 동작하기
tags: [zwave2mqtt, reverse proxy]
author: ilwoong
excerpt_separator: <!--more-->
---

zwave2mqtt의 주소를 reverse proxy의 proxy_pass에 입력하면 reverse proxy를 통해 접속되지 않습니다. 중간에 경로를 추가해줘야 하는데 ```X-External-Path``` 인자를 헤더에 넣어 해결할 수 있습니다.

<!--more-->

### /nginx/templates/default.conf.template 파일

```nginx
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    location /zwave/ {
        proxy_pass http://192.168.1.2:8091/;
        proxy_set_header X-External-Path /zwave; #location path로 설정함
    }
}

```

### 참고자료

- https://github.com/OpenZWave/Zwave2Mqtt/blob/master/docs/subpath.md
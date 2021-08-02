---
layout: post
title: GitHub push할때 ID/PW 입력 건너뛰기
tags: [GitHub, access token]
author: ilwoong
---

GitHub에서 최근 GitHub API나 커맨드창 명령어에서 패스워드 로그인 대신 access token을 이용하여 인증하도록 인증 방법을 변경하였습니다. 토큰을 발급받아 사용해보니, 커맨드창에서 push 명령을 수행할 때, 패스워드 대신 access token을 입력하기가 불편합니다. 길이가 길기도 하고 의미있는 문장이 아니라 외우기도 힘들어서, 별도로 저장해둔 다음 복사 및 붙여넣기를 해야 합니다. 귀찮아서 자동으로 입력하는 방법을 찾아봤습니다.

## GitHub access token 발급받기

Access token은 [다음 링크](https://docs.github.com/en/github/authenticating-to-github/keeping-your-account-and-data-secure/creating-a-personal-access-token){:target="_blank"}를 참고해서 발급받으면 됩니다.

## 프로젝트 설정 파일 수정하기

git으로 관리되는 폴더는 .git 라는 숨겨진 폴더가 있습니다. 해당 폴더 안에 config 파일이 존재하고, 이 파일 안에 다음과 같은 설정 내용이 있습니다.

```config
...

[remote "origin"]
        url = https://github.com/{username}/{reponame}
        fetch = +refs/heads/*:refs/remotes/origin/*

...
```

이 부분을 다음과 같이 수정하면 됩니다.

```config
...

[remote "origin"]
        url = https://{username}:{access_token}@github.com/{username}/{reponame}
        fetch = +refs/heads/*:refs/remotes/origin/*

...
```

## 주의사항

아이디 및 access token을 작업 폴더안에 평문으로 적어두는 것과 같기 때문에 반드시 개인 PC에서만 적용해야 합니다.
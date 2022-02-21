---
layout: post
title: WSL 배포판 여러 개 설치하기 
tags: [Windows 10, WSL]
author: ilwoong
---

WSL 위에서 동작하는 Ubuntu 배포판은 Windows Store에서 다운로드 받을 수 있습니다. 그러나 같은 버전의 배포판은 하나만 설치할 수 있는 제약사항이 있습니다. 용도를 달리해서 여러 배포판을 관리하고 싶을 때는 wsl 명령어를 사용해서 배포판을 설치해야 합니다.

### WSL 버전 설정

WSL2를 사용하기 위해서 Linux 커널 업데이트 패키지 설치가 필요할 수 있습니다. 아래 링크에서 패키지를 다운받아 설치합니다.

<https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi>

패키지 설치 후에 기본 WSL 버전을 2로 바꿔줍니다.

```powershell
PS:\> wsl --set-default-version 2
```

### 다운로드 & 압축 해제

<https://aka.ms/wslubuntu2004>

위 링크의 파일을 다운로드 받아 압축을 풉니다. 이 글에서는 d:\wsl\ubuntu2004 폴더에 압축을 풀었다고 가정합니다.


### 설치

다운로드 받아 압축을 해제한 배포판을 설치하고자 하는 위치에 설치합니다. 이 글에서는 설치할 위치를 d:\wsl\ubuntu 라고 가정합니다.

```powershell
PS:\> wsl --import ubuntu-my-dist d:\wsl\ubuntu d:\wsl\ubuntu2004\install.tar.gz
```

### 설치 확인

제대로 설치가 되었는지 확인하기 위해 현재 설치된 배포판의 목록을 출력해봅니다.

```powershell
PS:\> wsl -l -v
```

### 배포판 실행

특정 배포판을 지정해서 실행하기 위해서는 --distribution 또는 -d 옵션을 주고 wsl 명령어를 실행합니다. 이 때 bash shell을 실행하기 위해 맨 마지막에 bash 를 추가합니다.

```powershell
PS:\> wsl -d ubuntu-my-dist bash
```

### 사용자 추가

처음에는 root로 로그인 되므로 사용자를 하나 추가해야 합니다.

```bash
# 사용자 추가
root@ubuntu# useradd username 

# 패스워드 설정
root@ubuntu# passwd username 

# 편의를 위해 sudo 그룹에 사용자 추가
root@ubuntu# usermod -aG sudo username 
```

### 추가한 사용자로 로그인 하기

추가한 사용자로 로그인 하기 위해서는 --user 혹은 -u 옵션을 주어 사용자를 지정하여 wsl 명령어를 실행합니다.

```powershell
PS:\> wsl -d ubuntu-my-dist -u username bash
```

### 참고자료

- <https://godsman.tistory.com/entry/윈도우즈에-WSL-여러-개-설치하기>{:target="_blank"}
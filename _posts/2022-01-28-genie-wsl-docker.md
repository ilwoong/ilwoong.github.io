---
layout: post
title: Docker Desktop for Windows 없이 wsl2에서 docker 구동하기
tags: [wsl2, genie, docker]
author: ilwoong
---

WSL에서 Docker를 사용하기 위해서는 Docker Desktop이 설치되어 있어야 했습니다만, Docker Desktop이 유료화됨에 따라 회사에서는 Docker Desktop을 이용해서 WSL에서 Docker를 구동하기 힘들어졌습니다. 이번 포스트에서는 Docker Desktop 없이 WSL에서 genie를 이용하여 Docker를 구동하는 방법에 대해 설명해보겠습니다.

### 1-1. WSL2 설치하기: Windows 10 버전 2004 이상 또는 Windows 11의 경우

해당 버전의 윈도우의 경우 아래 명령어를 실행하면 wsl2가 자동으로 설치된다고 합니다. 단, wsl이 설치되지 않은 상태에서 실행해야 한다고 합니다.

```powershell
# powershell
wsl --install
```

### 1-2. WSL2 설치하기: Windows 10 버전 2004 미만

파워쉘을 열어 다음 명령어를 실행하여 WSL과 VM 기능을 활성화합니다.

```powershell
# powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

재부팅한 뒤 WSL2 Linux 커널 업데이트를 적용합니다.

- [x64 머신용 최신 WSL2 Linux 커널 업데이트 패키지 다운로드](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)


WSL의 기본 버전을 2로 설정합니다.
```powershell
# powershell
wsl --set-default-version 2
```

마이크로소프트 스토어에서 리눅스 배포판(Ubuntu 등)을 검색하여 설치합니다. 이 포스트에서는 Ubuntu를 사용합니다.

### 2. Docker 설치

WSL 리눅스 배포판에서 다음과 같이 docker와 docker-compose를 설치합니다.

```bash
# bash
sudo apt install docker.io docker-compose
```

docker 실행시마다 sudo를 실행하기는 귀찮기 때문에 권한을 부여합니다. {username} 부분은 실제 사용하는 유저 이름으로 변경하면 됩니다.

```bash
# bash
sudo usermod -aG docker {username}
```

이 상태에서 docker가 잘 실행되었는지 확인하기 위해 hello-world를 실행시켜봅니다.

```bash
# bash
docker run hello-world
```

docker daemon이 실행되지 않고 있으므로 아직 실행은 안됩니다.

```bash
docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?.
See 'docker run --help'.
```

### 3. .NET SDK 설치

systemd-genie를 설치하기 위해서는 .NET runtime이 필요합니다. .NET을 설치하기 전에 다음 명령을 실행하여 신뢰 키 목록에 Microsoft 패키지 서명 키를 추가하고 패키지 저장소를 추가합니다.

```bash
# bash
wget https://packages.microsoft.com/config/ubuntu/21.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb
```

.NET SDK를 설치합니다.

```bash
# bash
sudo apt-get update
sudo apt-get install -y apt-transport-https
sudo apt-get update
sudo apt-get install -y dotnet-sdk-6.0
```

### 4. systemd-genie 설치

systemd-genie는 WSL에는 없던 systemd를 WSL2에서 동작하도록 하는 프로젝트입니다. (WSL2만 지원합니다.)

systemd-genie를 설치하기 위해 저장소의 GPG Key를 받아옵니다.

```bash
# bash
sudo wget -O /etc/apt/trusted.gpg.d/wsl-transdebian.gpg https://arkane-systems.github.io/wsl-transdebian/apt/wsl-transdebian.gpg
```

저장소 리스트를 추가합니다.

```bash
# bash
sudo nano /etc/apt/sources.list.d/wsl-transdebian.list
```

이 파일에 다음과 같이 적어줍니다.

```bash
# wsl-transdebian.list
deb https://arkane-systems.github.io/wsl-transdebian/apt/ focal main
deb-src https://arkane-systems.github.io/wsl-transdebian/apt/ focal main
```

systemd-genie를 설치합니다.

```bash
# bash
sudo apt install systemd-genie
```

### 5. 설치 완료 테스트

기존에 실행하고 있던 wsl을 모두 종료합니다.

```powershell
wsl --shutdown
```

wsl을 실행할 때 genie를 실행하는 명령어를 추가하여 실행합니다.

```powershell
wsl genie -s
```

다음과 같은 메시지가 나오면서 실행되는데, 버그가 있는지 처음 실행하면 !가 계속 늘어나기만 하고 wsl이 실행되지 않습니다. ctrl + c를 눌러 종료한 다음 다시 실행해줍니다.

```powershell
Waiting for systemd....!!!!!!!!
```

이제 다시 docker에서 hello-world를 실행해봅니다.

```bash
docker run hello-world
```

다음과 같이 나오면 성공입니다.

```bash
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete
Digest: sha256:507ecde44b8eb741278274653120c2bf793b174c06ff4eaa672b713b3263477b
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

이제 Docker Desktop 없이 WSL에서 Docker를 사용할 수 있습니다.

### 참고자료

- [WSL 설치](https://docs.microsoft.com/ko-kr/windows/wsl/install){:target="_blank"}
- [이전 버전 WSL의 수동 설치 단계](https://docs.microsoft.com/ko-kr/windows/wsl/install-manual){:target="_blank"}
- [Setting Up Docker and docker-compose on WSL2](https://niwakatech.info/en/setting-up-docker-and-docker-compose-on-wsl2/){:target="_blank"}
- [genie](https://github.com/arkane-systems/genie){:target="_blank"}
- [Linux에 .NET 설치](https://docs.microsoft.com/ko-kr/dotnet/core/install/linux){:target="_blank"}
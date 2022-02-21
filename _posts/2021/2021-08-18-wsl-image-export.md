---
layout: post
title: WSL 이미지 백업 및 복원 하기
tags: [WSL, 이미지, 백업]
author: ilwoong
---

Windows Subsystem for Linux(WSL)은 윈도우에서 리눅스 쉘을 실행하는데 가장 편리한 도구입니다. 설정에서 WSL 항목을 활성화 시키고, 윈도우 스토어에서 원하는 리눅스 배포판을 설치하는 것으로 윈도우에서 리눙스 쉘을 사용할 수 있습니다. 그러나 윈도우를 새로 설치한다던가, 네트워크가 연결되지 않은 PC에 설치해야 할 필요가 있는 경우에는 미리 설정해둔 작업 환경을 매번 새로 설정하는 것은 매우 귀찮은 일입니다. 설치된 배포를 백업하고 복원하는 방법을 찾아봤습니다.

### 이미지 리스트 확인하기

윈도우 CMD 창을 열어서 다음과 같은 명령어로 현재 시스템에 설치된 배포의 리스트를 확인할 수 있습니다.

```powershell
wsl --list --all
```

### 이미지 내보내기

CMD 창에서 다음 명령어를 사용하여 리스트에서 확인한 배포를 원하는 경로에 tar 파일로 저장할 수 있습니다.

```powershell
wsl --export {DistroName} {FilePath.tar}
```

### 이미지 복원하기

CMD 창에서 다음 명령어를 사용하여 tar 파일로 저장된 이미지로부터 배포를 설치할 수 있습니다.

```powershell
wsl --import {DistroName} {DestinationFolderPath} {FilePath.tar}
```

### 이미지 삭제하기

CMD 창에서 다음 명령어를 사용하여 설치된 배포를 삭제할 수 있습니다.

```powershell
wsl --unregister {DistroName}
```

### 참고자료

- <https://websetnet.net/how-to-import-and-export-wsl-distros-on-windows-10/>{:target="_blank"}
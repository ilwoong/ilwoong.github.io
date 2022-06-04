---
layout: post
title: WSL2 포트 포워딩
tags: [wsl2, port forwarding]
author: ilwoong
excerpt_separator: <!--more-->
---

WSL2 에서 동작하는 네트워크 서비스는 로컬 머신에서는 접속이 가능하지만, 외부에서는 바로 접근이 불가능합니다. 외부에서 접속하기 위해 방화벽 설정이 필요합니다. 

<!--more-->

- 파워쉘 스크립트를 ```wsl-networks.ps1```와 같은 이름으로 작성합니다.

```powershell
# wsl-networks.ps1

# 관리자 권한으로 실행되는지 확인
If (-NOT ([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator"))
{   
  $arguments = "& '" + $myinvocation.mycommand.definition + "'"
  Start-Process powershell -Verb runAs -ArgumentList $arguments
  Break
}

# wsl ip 주소 확인
$wslAddr = bash.exe -c "ifconfig eth0 | grep 'inet '"
$found = $wslAddr -match '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}';

if ($found) {
  $wslAddr = $matches[0];
} else {
  echo "The Script Exited, the ip address of WSL 2 cannot be found";
  exit;
}

# 오픈할 포트 목록
$ports = @(80,443,10000,3000,5000);

# 외부로 오픈할 IP 주소 설정
$addr = '0.0.0.0';
$ports_a = $ports -join ",";

# 방화벽 룰 초기화
iex "Remove-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' ";

# 방화벽 룰 설정
iex "New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Outbound -LocalPort $ports_a -Action Allow -Protocol TCP";
iex "New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Inbound -LocalPort $ports_a -Action Allow -Protocol TCP";

for ($i = 0; $i -lt $ports.length; $i++) {
  $port = $ports[$i];
  iex "netsh interface portproxy delete v4tov4 listenport=$port listenaddress=$addr";
  iex "netsh interface portproxy add v4tov4 listenport=$port listenaddress=$addr connectport=$port connectaddress=$wslAddr";
}
```
- 관리자 모드로 파웨쉘 스크립트를 실행합니다. Exception Policy 경고가 발생하는 경우 아래와 같이 실행합니다.

```powershell
PowerShell.exe -ExecutionPolicy Bypass -File .\wsl-networks.ps1
```

이제 외부에서 WSL2에서 동작하는 네트워크 포트에 접근할 수 있습니다.

### 참고자료

- <https://gmyankee.tistory.com/308>{:target="_blank"}
- <https://github.com/microsoft/WSL/issues/4150>{:target="_blank"}
- <https://sungyong.medium.com/wsl2-port-forwarding-2f984a26c1fd>{:target="_blank"}
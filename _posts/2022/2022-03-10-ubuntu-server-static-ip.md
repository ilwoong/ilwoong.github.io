---
layout: post
title: Ubuntu 20.04 LTS 서버 고정 IP 설정하기
tags: [ubuntu, ip]
excerpt_separator: <!--more-->
---

Ubuntu 20.04 LTS 서버의 네트워크 설정 파일은 /etc/netplan 디렉토리 안에 있는 yaml 파일입니다.

<!--more-->


DHCP로 설정된 경우 아래와 같이 되어 있습니다.

```yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    enp0s3:
      dhcp4: true
  version: 2
```

다음과 같이 변경하고,

```yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    enp0s3:
      addresses: [192.168.0.12/24]
      gateway4: 192.168.0.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
  version: 2
```

설정 변경 사항을 적용합니다.

```bash
netplan apply
```

### 기타

네트워크 테스트를 위해 연결이 되지 않은 상태로 부팅하는 경우 아래와 같이 네트워크 연결을 기다리는 프로세스를 비활성화 시킵니다.

```bash
sudo systemctl disable systemd-networkd-wait-online.service
sudo systemctl mask systemd-networkd-wait-online.service
```

### 참고자료
- manualfactory.net/13079
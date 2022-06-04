---
layout: post
title: 라즈베리파이 기본계정 pi 변경
tags: [raspberry pi]
author: ilwoong
excerpt_separator: <!--more-->
---

라즈베리파이 OS를 설치하면, 기본 계정이 pi/raspberry로 설정되어 있어, 2021년 발표된 [OWASP TOP 10의 5번째 항목](https://owasp.org/Top10/A05_2021-Security_Misconfiguration/)에 해당하는 기본 계정/패스워드를 그대로 사용하는 취약점이 존재합니다. 이 취약점을 없애기 위해 기본 계정을 아래와 같이 변결할 수 있습니다. **최신 버전의 Raspberry Pi OS는 설치 시에 계정 이름 및 패스워드를 입력하게 변경되었습니다.**

<!--more-->

### 1. 임시 계정 생성

```bash
adduser {계정이름}
usermod -aG sudo {계정이름}
```

### 2. 임시 계정으로 로그인

```bash
su {계정이름}
```

### 3. pi 계정 변경

```bash
usermod -l {새 계정이름} pi
usermod -m -d /home/{새 계정이름} {새 계정이름}
```

### 4. 임시계정 삭제

```bash
userdel -r {계정이름}
```

### 5. 변경내용 확인

```bash
users
```
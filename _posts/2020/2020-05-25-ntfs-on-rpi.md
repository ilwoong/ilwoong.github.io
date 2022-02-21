---
layout: post
title: 라즈베리파이에서 NTFS 외장하드 사용하기
tags: [Raspberry Pi, NTFS]
author: ilwoong
---

윈도우 PC에 연결해서 사용하던 NTFS 파일시스템으로 포맷된 외장하드를 라즈베리파이에서 사용하고 싶어졌습니다. 라즈베리파이4가 USB3.0을 제대로 지원하기 시작했다고 하고, 일반 PC에 붙여놓는 것 보다 전력을 적게 사용할 것으로 기대가 됩니다.

인터넷을 뒤져 필요한 패키지가 무엇인지 찾아서 설치합니다. 

```bash
$ sudo apt install ntfs-3g fonts-unfonts-core
```

- ntfs-3g: NTFS 하드디스크에 쓰기 작업을 하기 위해 필요한 패키지
- fonts-unfonts-core: 한글 출력을 위한 폰트

외장하드를 라즈베리파이의 USB에 연결하고 전원을 넣은 다음 잘 인식되었는지 확인하기 위해 lsblk 명령어를 사용합니다. 제가 사용한 외장하드 케이스 [Sabrent DS-4SSD](https://www.sabrent.com/product/DS-4SSD/usb-3-0-4-bay-2-5-hard-drive-ssd-docking-station-fan){:target="_blank"}는 4개의 2.5인치 하드디스크를 지원하는 제품입니다. 하드디스크를 4개나 사용하지만, 별도의 전원 어댑터를 사용하는 제품이라 라즈베리파이에서 전원공급을 고민하지 않아도 됩니다.


```bash
$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda           8:0    0  1.8T  0 disk
└─sda1        8:1    0  1.8T  0 part
sdb           8:16   0  1.8T  0 disk
├─sdb1        8:17   0  128M  0 part
└─sdb2        8:18   0  1.8T  0 part
sdc           8:32   0  1.8T  0 disk
└─sdc1        8:33   0  1.8T  0 part
sdd           8:48   0  1.8T  0 disk
├─sdd1        8:49   0  128M  0 part
└─sdd2        8:50   0  1.8T  0 part
mmcblk0     179:0    0 59.6G  0 disk
├─mmcblk0p1 179:1    0  256M  0 part /boot
└─mmcblk0p2 179:2    0 59.4G  0 part /
```

4개의 하드디스크가 모두 잘 인식되었습니다. 

일회성으로 마운트를 하고 싶은 경우는 다음과 같이 mount 명령어를 이용합니다.

```bash
$ sudo mount -t ntfs /dev/sda1 /mnt/ext1
```

마운트 후 df 명령어로 잘 마운트 되었는지 확인할 수 있습니다. 이제 /mnt/ext1이라는 폴더로 외장하드를 접근할 수 있습니다.

```bash
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/root        59G  3.3G   53G   6% /
devtmpfs        1.8G     0  1.8G   0% /dev
tmpfs           1.9G     0  1.9G   0% /dev/shm
tmpfs           1.9G  8.5M  1.9G   1% /run
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
tmpfs           1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/mmcblk0p1  253M   53M  200M  21% /boot
tmpfs           376M     0  376M   0% /run/user/1000
/dev/sdc1       1.9T  500G  1.4T  27% /mnt/ext1
```

사용이 끝난 후 언마운트를 하고 싶으면 umount 명령어를 사용합니다.

```bash
$ sudo umount -v /dev/sda1
```

매번 마운트하는 것은 번거롭습니다. /etc/fstab 파일에 다음 항목을 수정하고 재부팅 하면 자동으로 마운트가 됩니다.

```bash
# <filesystem> <mount point> <type> <options> <dump> <pass>
UUID=1AAC1487AC146015 /mnt/ext1 ntfs default,nofail 0 0
```

- filesystem: 마운트할 파일 시스템 (/dev/sda, UUID=... 등)
- mount point: 마운트할 경로
- type: 파일시스템 이름 (ext4, ntfs 등)
- options: 마운트 옵션 (rw, ro, noexec 등)
- dump: 파일시스템이 dump가 필요한지 여부 설정 (0, 1)
- pass: 리부팅 시 파일 시스템을 검사할지 여부 설정 (0, 1)

참고로 UUID는 blkid 명령어로 확인할 수 있습니다.

```bash
$ blkid
/dev/mmcblk0p1: LABEL_FATBOOT="boot" LABEL="boot" UUID="6341-C9E5" TYPE="vfat" PARTUUID="ea7d04d6-01"
/dev/mmcblk0p2: LABEL="rootfs" UUID="80571af6-21c9-48a0-9df5-cffb60cf79af" TYPE="ext4" PARTUUID="ea7d04d6-02"
/dev/sdb2: LABEL="ext1" UUID="C6DA911EDA910C35" TYPE="ntfs" PARTLABEL="Basic data partition" PARTUUID="cb52e136-d449-4106-9ba0-4a8cbc8e4e65"
/dev/sdc1: LABEL="ext2" UUID="1AAC1487AC146015" TYPE="ntfs" PARTUUID="bec865cd-01"
/dev/sda1: LABEL="ext4" UUID="C41EDDD21EDDBD9C" TYPE="ntfs" PARTUUID="2ef8bb5d-01"
/dev/sdd2: LABEL="ext3" UUID="104ABFB44ABF94C6" TYPE="ntfs" PARTLABEL="Basic data partition" PARTUUID="9596b51b-c02b-434b-aa76-3355cfc3534e"
```

### 참고자료
- <https://geeksvoyage.com/raspberry%20pi/sdcard-for-pi/>{:target="_blank"}
- <https://dev.meye.net/entry/fstab>{:target="_blank"}
---
title: "Kali linux: Mac에서 booting USB만들기"
excerpt_separator: <!--more-->
categories:
  - Mac
  - Kali
  - linux
tags: 
  - mac
  - kali
  - linux
---

사용 중이던 `kali linux`의 의존성 패키지를 설치하다가, 패키지를 잘못 건드려서 `GUI booting`이 안되는 대참사가 일어났다. 주말을 꼬박 써서 해결해보려 했으나.. 포기하고 다시 깔기로 하였다.  칼리 세팅에 투자한 시간이 너무 아깝다만 별 수 없었다.

부팅 USB를 만들어야하는데,  사실 Window에서만 해보고 Mac에서 처음이다. 한번 해보자.
<!--more-->
1. 먼저 `ISO`이미지를 다운받는다. 각자 필요한 이미지를 받으면 되겠다. 나 같은 경우는 [칼리리눅스](https://www.kali.org/downloads/) 64bit 이미지를 받았다. 

2. `Terminal`에서 `hdiutil Convert`한다.
```bash
$ hdiutil convert -format UDRW -o ~/path/to/kali-linux.img  ~/path/to/kali-linux-2018.3a-amd64.iso
Master Boot Record(MBR : 0) 읽는 중...
Kali Live                       (Apple_ISO : 1) 읽는 중...
(Windows_NTFS_Hidden : 2) 읽는 중...
......................................................................................................................................................
(DOS_FAT_12 : 3) 읽는 중...
......................................................................................................................................................
경과 시간: 13.743s
속도: 221.5Mbytes/초
저장: 0.0%
created: ~/path/to/kali-linux.img.dmg
```

3. USB가 몇 번에 마운팅되어 있는 확인한다. HIDEKUMA USB가 `/dev/disk2`에 맵핑되어있는 걸 확인했다. (나중에 엉뚱한데에 쓰면 답이 없다. 잘 확인한다.)
```bash
$ diskutil list
/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *8.0 GB     disk2
   1:                 DOS_FAT_32 HIDEKUMA                720.9 KB   disk2s1
```

4. `unmount`시킨다.
```bash
$ diskutil unmountDisk /dev/disk2
Unmount of all volumes on disk2 was successful
```
5. 이미지를 복사한다. USB 2.0 기준약 20분(8GB) 정도 소요되었다.
```bash
$ sudo dd if=~/path/to/kali-linux.img.dmg of=/dev/disk2 bs=1m
Password:
3044+1 records in
3044+1 records out
3192651776 bytes transferred in 1252.474807 secs (2549075 bytes/sec)
~ 20m 56s
```

6. 드라이브를 꺼낸다. 완성.
```bash
$ diskutil eject /dev/disk2
Disk /dev/disk2 ejected
```

7. USB 초기화 할 경우.
```bash
$ sudo dd if=/dev/zero of=/dev/disk2 bs=1m
```
초기화 시에는, 상기 커맨드로 초기화 후 디스크 유틸리티에서 지우기를 진행하면 된다.

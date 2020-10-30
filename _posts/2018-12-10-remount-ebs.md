---
layout: post
title: "Linux: Ubuntu16.04: EC2: 어느날 갑자기 Read-only file system 으로 변해버렸을때"
excerpt_separator: <!--more-->
categories:
  - EC2
  - Ubuntu
  - Linux
tags: 
  - file system
---

### Read-only file system ERROR?
```bash
$ mkdir test
mkdir: cannot create directory 'test': Read-only file system
```
잘 쓰던 EC2 인스턴스에서 이러한 에러가 발생했다. 보자마자 하기 이유들이라고 예상했다.
1. 디스크용량
2. 아이노드 용량
3. 그 외

**BUT...**
1.  디스크/아이노드 용량
```bash
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/nvme0n1p1  30G   7.7G 22G   27%  /
$ df -i
Filesystem      Inodes  IUsed   IFree IUse% Mounted on
/dev/nvme0n1p1 3840000 252936 3587064    7% /
```
어라, 정상이다.
3번째 가능성은 저 `Read-only file system`부분이다.
<!--more-->

2. 파일시스템
```bash
$ cat /proc/mounts
/dev/nvme0n1p1 / ext4 ro,relatime,discard,data=ordered 0 0
```
`ro`로 되어있다. 마운팅 권한이 이렇게 되어버린 정확한 이유는 못찾았으나, 해당 부분이 상기 에러를 발생하는것임이 틀림없었다.
따라서, 권한을 `rw`로 변경하기로 하였다.

3. 리마운팅
```bash
$ sudo umount /dev/nvme0n1p1
$ sudo e2fsck /dev/nvme0n1p1
$ sudo mount -o remount, rw /
$ cat /proc/mounts
/dev/nvme0n1p1 / ext4 rw,relatime,discard,data=ordered 0 0
```
`rw`로 변경되었다. 정상적으로 디렉토리도 생성된다. 구글링에 의하면, 가끔 디스크유틸이 튀거나 오류를 발생할 때 권한이 `ro`로 바뀐다고 한다.
- umount
: 체크 할 파티션을 umount한다(필수). 파티션 손실 가능성을 최소화하기위함.
- e2fsck
: 리눅스 파일시스템 체크/복구 툴(inodes, blocks, sizes, directories, links, file counts)

4. 정상화
![image-center]({{ '/images/post-imgs/remount-error-solved.png' | absolute_url }}){: .align-center}

도대체 12/09 무슨일이 있었던거지...
그리고 또 한가지, SSH접속을 끊고 재접속을 하려고하니, 접속이 안되었다. AWS 콘솔에서 해당 EC2를 리부팅해주고나서야 SSH접속이되고 정상화되었다.

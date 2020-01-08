---
title: "Windows: python not working in the command line of git bash"
excerpt_separator: <!--more-->
categories:
  - Python 
  - Gitbash  
  - Winpty
tags: 
  - Python 
  - gitbash
  - winpty
---

# gitbash + python
gitbash에서 python을 인스톨 후, 이제 사용해보려고 python 커맨드를 치는 순간.
```bash
$ python
```
위 상태로 묵묵부답이다.

<!--more-->

# winpty 
다행히도 winpty로 해결 가능하다. winpty는 Windows 콘솔 프로그램과 통신하기 위해 Unix 환경과 유사한 인터페이스를 제공하는 패키지로 python 커맨드를 다음과 같이 입력하면 정상적으로 동작한다.
```bash
$ winpty python.exe
Python 3.7.0 (default, Oct  2 2018, 09:18:58)
Type "help", "copyright", "credits" or "license" for more information.
>>>
```
그러나 매번 저렇게 치긴 귀찮으니, `.bashrc`에 기입해주면 된다.
```bash
$ echo "alias python='winpty python.exe'" >> ~/.bashrc
```

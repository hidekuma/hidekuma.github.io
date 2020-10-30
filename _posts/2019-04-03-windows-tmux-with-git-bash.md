---
layout: post
title: "Gitbash: gitbash에서 tmux 사용하기"
excerpt_separator: <!--more-->
categories:
  - Windows
  - Gitbash
  - Tmux
tags: 
  - windows
  - gitbash
  - tmux
---

# Windows + git-bash + tmux
나는 linux기반인 MacOS 찬양자였으나, 이번 이직으로 윈도우 환경에서 일을 하게 되었다. WindowOS에서는 Bash사용이 불편할 수 밖에 없는데, 별 수 없다. 환경이야 만들면 된다.

## git-bash
다행히 어렵지 않았다. cmder나 powershell도 많이 사용하는 것 같으나, git설치만으로 같이 설치되는 git-bash가 사용도 편리해서 이걸로 정했다. 평소에 터미널에서 tmux를 사용하는데, git-bash에 반영하는데 사실 몇 가지 문제가 있어서 해당 내용을 블로깅 해보려고한다. git은 공홈에서 다운받아 사용하면 된다.

<!--more-->
## git-bash + tmux
1. msys2 설치
  - https://www.msys2.org/에 접속하여, 메뉴얼대로 msys2를 설치한다. msys2는 Windows 용 소프트웨어 구축 플랫폼이다. 
2. pacman을 통해 tmux를 인스톨한다.
  - msys2 콘솔에서 pacman이라는 패키지 관리툴을 사용할 수 있다.
  - ```bash
$ pacman -S tmux
```
3. 인스톨이 완료되면, 파일탐색기로 `C:\msys64\usr\bin`에 이동한다. (만약 설치 시, 경로를 바꿨다면 해당 경로하위 bin 폴더로 이동)
4. `tmux.exe` 와 `msys-event-2-x-x.dll` 를 복사한다.
5. `C:\Program Files\Git\usr\bin` (git 경로)의 bin 디렉토리에 붙여넣기 해준다.
6. git-bash에서 이제 tmux를 이용할 수 있다.

**업데이트**<br/> msys-event가 버전업 되거나 새로운 tmux배포판을 설치 할 경우, 번거롭지만 동일한 작업을 해줘야한다.
{: .notice--info}

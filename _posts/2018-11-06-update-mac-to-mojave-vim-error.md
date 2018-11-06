---
title: "MacOS: 맥에서 Mojave(모하비) 업데이트 후 vim 에러"
excerpt_separator: <!--more-->
categories:
  - Mac
  - MacOS 
  - Mojave 
  - Vim
tags: 
  - macOS
  - vim
---

### 맥 모하비 업데이트를 했느데 이게 왠걸.. vim이 에러뜨면서 꺼진다.
**Exception MemoryError: MemoryError() in module threading from python**

모하비 업데이트시, 애플에서 `vim`을 강제 컴파일 하는걸로 알고있는데, 복불복인가. 주변 동료는 문제없이 업데이트 되었는데, 나만 해당에러가 뿜뿜하면서 꺼졌다.
`brew`로 `vim`을 새로 설치하고, `alias vi=vim`으로 해줘도 해결할 수도 있다.

그러나, `git`에서 `system vim`을 보고있기때문에, 소스 병합시 커밋메세지를 보낼 때 에러가 뜬다.
구글링하다보니 나와 같은 현상의 사람들이 종종 보이는것 같다. 그래서 해결법을 공유하고자한다.

<!--more-->
### 해결법
1. 버전을 한번 확인해보자.
```bash
$ vi --version
VIM - Vi IMproved 8.0 (2018 May 18, compiled Nov  6 2018 11:52:26)
macOS version
Included patches: 1-500
Compiled by root@apple.com
```
`root@apple.com`에 의해 컴파일 된것을 확인할 수 있다.

2. 먼저 brew로 설치한 vim이 있다면 지워주자.
```bash
$ brew uninstall vim
```

3. vim을 설치하는데 시스템 vim을 덮어쓰기한다.
```bash
$ brew install vim --with-lua --with-override-system-vi
```

4. 터미널이나 쉘을 다시 실행시켜보면 정상적으로 동작한다.
나처럼 고생하는 일이 없으면 좋겠다.

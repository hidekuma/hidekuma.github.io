---
layout: post
title: "Pyenv: pyenv 인스톨이 실패할 경우 (No module named 'pyexpat')"
excerpt_separator: <!--more-->
categories:
  - python
  - pyenv
tags:
  - pyexpat
---

# pyexpat이 원인으로 pyenv install 이 실패할 경우
`xcode`가 업데이트만 하면 항상 이런문제가 생기는 것같다.

```bash
# 잘못된 버전이 위치된 CLI 툴을 삭제한다.
$ sudo rm -rf /Library/Developer/CommandLineTools
# 다시 설치한다.
$ xcode-select --install
# 설치 확인
$ pkgutil --pkg-info=com.apple.pkg.CLTools_Executables
package-id: com.apple.pkg.CLTools_Executables
version: 10.3.0.0.1.1562985497
volume: /
location: /
install-time: 1570155487
groups: com.apple.FindSystemFiles.pkg-group
```

여러가지 방법을 시도해봤지만, 재설치이외에는 해결법이 없었다.

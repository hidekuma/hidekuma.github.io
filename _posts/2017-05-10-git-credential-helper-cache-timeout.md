---
title: "Git: 계정 password에 timeout 물리기"
excerpt_separator: <!--more-->
categories:
  - Git
  - Credential
tags: 
  - git
  - credential
  - helper
  - cache
---

## Git에서 계정 비밀번호를 저장 혹은 캐싱하자.

`Git` 사용자 중에서도 의외로 모르는 사람들이 있는 것같아 공유하고자한다. 우리가 `Git`을 사용하다 보면 `git push`나 `git pull`등을 할 때, 비밀번호를 입력해야하는데 해당 부분을 없애거나 줄일 수 있다. 

1. Repository URL에 직접 입력.
보통 프로젝트를 `git clone`하게 되면, 프로젝트 내에 `.git/config`가 생긴다. 해당 `config`를 수정해주면 되는데, 깃 커맨드도 있지만 나는 `vim`으로 직접 수정하는 편이다.
<!--more-->
```bash
$ cat .git/config
[core]
    repositoryformatversion = 0
    filemode = true
    bare = false
    logallrefupdates = true
[remote "origin"]
    url = https://github.com/hidekuma/hidekuma.github.io.git         
    fetch = +refs/heads/*:refs/remotes/origin/*                               
[branch "master"]
    remote = origin
    merge = refs/heads/master
```
이런식으로 `config`가 설정되어 있을텐데, 여기서 `[remote "origin"] > url`을 변경해주면 된다. 
```bash
[remote "origin"]
    url = https://아이디:패스워드@github.com/hidekuma/hidekuma.github.io.git         
```
허나 해당 방법은 다소 보안상 위험하다. 따라서 나같은 경우, 아이디만을 넣어준다.
```bash
[remote "origin"]
    url = https://아이디@github.com/hidekuma/hidekuma.github.io.git         
```

2. 캐시설정.
위에 제시된 방법처럼, `remote url`에 아이디만 넣어주면, 깃 커맨드를 사용할 때마다 계속해서 비밀번호를 물어올것이다. 생각만 해도 끔찍한데, 이 빈도수를 확 줄일수가 있다.
```bash
$ git config --global credential.helper cache 
```
이렇게 하면 15분간 비밀번호를 캐싱해준다. 디폴트가 900초인 셈인데, 아직도 짧다.
```bash
git config --global credential.helper 'cache --timeout=432000'
```
이렇게 하면 타임아웃 시간을 커스텀할 수 있다. 자 이제 5일간 캐싱을 해준다. 살았다.

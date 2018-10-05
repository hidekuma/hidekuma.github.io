---
title: "Python: 가상환경 만들기/ virtualenv + virtualenvWrapper"
categories:
  - Python
  - Virtualenv
  - VirtualenvWrapper
tags: 
  - virtualenv
  - virtualenvWrapper
---

## Python 가상환경을 손쉽게 구축하자.
Python으로 프로젝트를 진행하는데, 가상환경을 구축하는 건 필수적이다. `venv`, `pyenv` 등 가상환경을 관리해주는 패키지들이 있는데, 그 중에서도 `virtualenv`와 `virtualenvWrapper`를 가장 좋아한다. 이유는 개인적으로 가상환경 구축과 동시에 관리 등이 가장 편리했다고 느꼈기 때문이다.

한번 설정해보자.

---
### virtualenv 설치
Mac에는 이미 python2가 설치되어있지만, python3를 이용하고싶기 때문에 먼저 설치다. `pyenv`가 설치되어있다면, pyenv를 사용하는게 바람직하다.

1. python3 설치
```bash
$ brew install python3
$ python -m pip -V
pip 9.0.1 from /Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages (python 3.6)
```

2. 정상적으로 설치가 완료되었다면, 패키지들을 설치하자.
```bash
$ pip3 install virtualenv virtualenvwrapper
```

3. 가상환경 만들기
```bash
$ virtualenv --python=python3.6 [가상환경이름]
```

4. 가상환경 진입하기
```bash
$ source [가상환경이름] /bin/activate
```

5. 가상환경 벗어나기
```bash
$ deactivate
```

만약 The path x.x does not exist라는 에러가 난다면 PYTHON의 PATH 경로문제이다. 환경변수를 절대경로로 맞춰주면 된다.
예를들어, which python3을 했을 때 /usr/bin/python3이 나왔다면, virtualenv --python=/usr/bin/python3와 같이 절대경로로 입력해주시면 해결된다.
{: .notice--warning}

---

### virtualenvWrapper 설치
이렇게 `virtualenv`를 통해서 가상환경을 만들고 진입하는 방법을 알아보았다. 하지만 불편한점이 있다. `source`를 통해 가상환경에 진입하려면 설치된 디렉토리로 이동하고, 어디에 가상환경이 구축되어있는지 사용자가 일일이 알고 확인해야하기 때문. 이러한 문제점을 해결하기 위해  `virtualenvWrapper`을 사용하자.

1. 가상환경 디렉토리 설정
```bash
$ mkdir ~/.virtualenvs
```

2. 환경변수 설정하기
```bash
$ export WORKON_HOME=~/.virtualenvs
$ export VIRTUALENVWRAPPER_PYTHON='파이썬의 경로'  # Usage of python3
$ source /usr/local/bin/virtualenvwrapper.sh
```
home 디렉토리에 `.bashrc`나 `.bash_profile`의 마지막에 상기 코드를 복붙해준다. 파일이 없다면 만들면 된다. 또는 zsh를 사용중일 경우에는 .zshrc쪽에 넣어주면 된다. python의 경로를 모를경우에는 `which python3`로 검색해보자.

3. 혹시 `/usr/local/bin`에서 `virtualenvwrapper.sh` 파일을 찾을 수 없다면, `/usr/bin`에 있을 수 있다. 잘 모르겠으면 `find`로 찾아보자. 
```bash
$ find /usr -name virtualenvwrapper.sh
```

4. 설정이 완료되었다면, terminal을 다시 키거나 `source ~/.bashrc`로 설정을 재로드 해준다.

5. 커맨드는 다음과 같다. 
```bash
$ mkvirtualenv [가상환경이름]
$ rmvirtualenv [가상환경이름]
$ workon [가상환경이름]
$ setvirtualenvproject
$ cdproject
$ deactivate
```

---
### 자주쓰는 commands

mkvirtualenv [가상환경이름]
: 가상환경이름으로 가상환경을 만들어준다. `mkvirtualenv test -p [python-path]`로 python 버전을 지정해 줄 수 있다.

rmvirtualenv [ 가상환경이름]
: 해당 가상환경을 지운다.

workon [가상환경이름]
: 해당 가상환경으로 진입한다.

servirtualenvproject
: 프로젝트 디렉토리로 이동 후, 해당 커맨드를 치면 디렉토리와 가상환경이 맵핑된다. 그러면 `workon`커맨드를 치는것 만으로 해당 디렉토리에 이동까지 시켜준다. 아주좋다.

cdproject
: `setvirtualenvproject`를 했을 경우, 현재 어디에 있든 해당 커맨드로 프로젝트 디렉토리에 이동이 가능하다.

deactivate
: 가상환경에서 빠져나온다.
 









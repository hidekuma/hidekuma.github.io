---
layout: post
title: "Python: WSL에서 mataplotlib show 사용세팅"
excerpt_separator: <!--more-->
categories:
  - Python 
tags: 
  - python 
  - anaconda 
  - miniconda
  - wsl
  - tensorflow 
  - window
  - subsystem
  - linux
---

# matplotlib with WSL
데이터 시각화 툴인 `mataplotlib`는 머신러닝 분야에서 배제 할 수없는 툴이다. 윈도우의 `WSL` 안에 `miniconda`를 설치하고 `tensorflow`를 테스트 해보던 도중에, `matplotlib`의 `show`가 적용되지 않는 문제가 있었다. 해결법은 다행히 간단하다.

<!--more-->
## 해결법
1. [Xming(X 윈도우 서버)](https://sourceforge.net/projects/xming/)를 설치한다.
2. `putty`는 필요해따라 설치하거나 한다.
3. ```bash
sudo apt-get update
sudo apt-get install python3.{version}-tk
conda install matplotlib
export DISPLAY=localhost:0.0 // .bashrc에 넣는게 좋음
```
이렇게 하면, `Xming`을 통해 데이터 시각화가 정상적으로 동작하는 것을 확인할 수 있다.

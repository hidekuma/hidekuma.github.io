---
layout: post
title: "Python: pycurl이 설치가 안되거나, import안될 경우 (No module named pycurl)" 
categories:
  - Python
  - Pycurl
tags: 
  - pycurl
---

EC2서버를 세팅중인데, pycurl이 설치가 되었는데도 불구하고, import가 안된다. 길러왔던 구글링 실력으로 찾아보니 비슷한 현상이 꽤나 있었으나 애매한 답변뿐이였다. 일단은 급하게 프로젝트를 진행해야하니 자세하게 뜻을 알아보는건 나중으로 미루고 해결방법만 공유하고자 한다.

1. 설치 된 pycurl을 지운다.
2. 하단 코드로 다시 설치한다. pycurl의 SSL library 환경설정을 해주는 커맨드다.
```bash
export PYCURL_SSL_LIBRARY=openssl
pip install pycurl --global-option=build_ext --global-option="-L/usr/local/opt/openssl/lib" --global-option="-I/usr/local/opt/openssl/include"
```
3. 아직도 안된다면
```bash
sudo env ARCHFLAGS="-arch x86_64" easy_install setuptools pycurl==7.19.0
```
버전은 최신꺼 확인해서 넣어보면 된다.
4. 보통 이걸로 해결이 되더라. 만약 ubuntu일 경우.
```bash
sudo apt-get install libcurl4-openssl-dev
```

안된다면... 그냥 requests로 갈아타자. 이 좋은걸 냅두고 고생할 필요없다.

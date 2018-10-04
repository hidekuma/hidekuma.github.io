---
title: "Python: pyenv를 통한 다양한 파이썬 버전 지정법"
categories:
  - Python
  - Pyenv
tags: 
  - pyenv
---

### 단순히 Python 버전을 지정하는 방법
pyenv에서 python 버전을 관리해주는데요. 단순히 python 라고 쳤을 때의 버전을 지정하고싶다면!

```python
pyenv global 2.7.5
```
라고 하면 됩니다.

### 하지만, python2.7 이라던지 python3.5 처럼 세세하게 사용하고 싶다면?
예를 들어, python3.5를 쳤을 때 이런 오류가 나옵니다.

```
pyenv: python3.5: command not found
The 'python3.5' command exists in these Python versions:
  3.5.2
``` 

이럴 때는 여러 버전을 나란히 쓰면됩니다. 물론 pyenv install [python-version] 으로 해당 버전의 파이썬이 인스톨되어있어야겠죠?
```python
  pyenv global system 2.7.5 3.5.2 3.3.2
```
  
이렇게하면 기본 python에서는 시스템 python이 사용되어 python2.7에서는 2.7.5, python3.5에서는 3.5.2, python3.3에서는 3.3.2가 사용된다는 식으로 됩니다!

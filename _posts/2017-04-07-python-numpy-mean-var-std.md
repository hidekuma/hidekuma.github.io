---
title: "Python: 파이썬으로 표준편차, 평균, 분산 구하기"
categories:
  - Python
  - Numpy
tags: 
  - 분산
  - 표준편차
  - 평균
---

평균[^1], 분산[^2], 표준편차[^3]의 뜻을 다들 알고 있기 때문에 이 곳에 도달했다고 가정하겠다. 단순하게 답만 보고가자.

[^1]: <https://ko.wikipedia.org/wiki/평균>
[^2]: <https://ko.wikipedia.org/wiki/분산>
[^3]: <https://ko.wikipedia.org/wiki/표준편차>

## Answer
```python
import numpy
arr = [1, 2, 3]

numpy.mean(arr) # 평균
# 2.0
numpy.var(arr) # 분산
# 0.66666666666666663
numpy.std(arr) # 표준편차
# 0.81649658092772603

```

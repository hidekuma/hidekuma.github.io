---
title: "Python: 평균, 분산, 표준편차 구하기"
sub_title: "The common elements"
excerpt: "파이썬으로 평균, 분산, 표준편차 구하기"
categories:
  - python
	- numpy
last_modified_at: 2018-02-01T10:16:49-05:00
---

## Answer

평균, 분산, 표준편차의 뜻을 다들 알고 있기 때문에 이 곳에 도달했다고 가정하겠다. 단순하게 답만 보고가자.

```python
import numpy
arr = [1, 2, 3]

numpy.mean(arr) # 평균
numpy.var(arr) # 분산
numpy.std(arr) # 표준편차
```




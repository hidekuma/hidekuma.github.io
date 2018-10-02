---
title: "Python: 파이썬으로 표준편차, 평균, 분산 구하기"
categories:
  - Markup
elements:
  - content
  - css
  - formatting
  - html
  - markup
---

평균, 분산, 표준편차의 뜻을 다들 알고 있기 때문에 이 곳에 도달했다고 가정하겠다. 단순하게 답만 보고가자.

## Answer

```python
import numpy
arr = [1, 2, 3]

numpy.mean(arr) # 평균
numpy.var(arr) # 분산
numpy.std(arr) # 표준편차
```




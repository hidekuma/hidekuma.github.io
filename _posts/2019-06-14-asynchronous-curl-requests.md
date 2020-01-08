---
title: "Python: asynchronous cURL requests: AsyncURL"
excerpt_separator: <!--more-->
categories:
  - Python
tags: 
  - asynchronous 
  - curl 
  - request 
  - asyncurl 
  - library 
  - uvloop 
  - asyncio 
---

# 비동기 cURL 리퀘스트 라이브러리 asyncurl
최근 파이썬에 유명한 비동기 프레임워크 `sanic`을 살펴보다가, 내부 이벤트 루프를 `uvloop`로 대체하여 비동기통신을 빠르게 했다는 [벤치마킹 결과](/python/uvloop/)를 감명깊게 읽었다.
결과적으로 `asyncio`의 퍼포먼스를 2배 이상 상승시킨 결과를 보고, 몇년 전 프로젝트에서 만들었었던 웹 크롤러를 떠올렸다.

파이썬의 `GIL`이라는 녀석 때문에 당시에는 멀티프로세싱을 이용하여 병렬처리를 구현했었는데, 이번에는 하나의 스레드로 병렬처리를 구현하는 비동기 라이브러리를 구현하기로 마음 먹었다.

## 의존성
파이썬 3.6 이상부터 지원하며, 작성일 기준 상세 라이브러리 의존성은 다음과 같다.

| Package  | Version  | Description           |
| :-:      | :-:      | :-:                   |
| asyncio  | >=3.4.3  | Asynchronous          |
| requests | >=2.22.0 | pycurl substitutes    |
| uvloop   | >=0.12.2 | for event loop policy |

이벤트루프 대체제인 `uvloop`, 비동기 라이브러리 `asyncio`, 마지막으로는 통신을 위한 `requests` 이상 3가지이다.
개인적으로 `pycurl`은 설치 의존성도 까다롭고, 사용과 설치에 귀찮았던 기억들이 있어서 사용성 면에서 갑인 `requests`를 택하였다.

<!--more-->

## Concept
`asyncio`와 `requests`만 있다면, 누구든지 비동기 cURL 리퀘스트를 구현할 수 있다.
하지만 해당 라이브러리를 이해하고 코루틴 짜는데 리소스가 소요되기 때문에, 그런 부분을 해소하기 위해 만든것이 [AsyncURL](https://pypi.org/project/asyncurl/)이다. 
조금 다른 점이라고 하면, 내부 이벤트 루프를 `asyncio`의 내장 이벤트 루프가 아닌, `uvloop`의 이벤트 루프 정책을 사용했단 점이다.

## Installation
배포판은 [여기서 다운로드](https://github.com/hidden-function/asyncurl/releases) 할 수 있다.

### Using pip
```python
pip install asyncurl
```

## Usage 
사실 구체적인 사용법이나 내용은 [README.md](https://github.com/hidden-function/asyncurl)에서 확인할 수 있고, 이번 포스팅에서는 예시만 살짝 공유하고, 깊게 다루진 않겠다.
```python
from asyncurl.session import AsyncURLSession
from asyncurl.fetch import AsyncURLFetch

ac_fetch = AsyncURLFetch()

for x in range(10):
    session = AsyncURLSession()
    session.fetch_url = 'http://localhost' 
    session.fetch_method = 'POST'
    session.headers = {'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36'}
    ac_fetch.queue.put_nowait(session)

ac_fetch.parallel()
```

## 회고 
사실 뭐 아직 베타 정도의 개발 단계이기 때문에, 개선해야 할 여지가 많다. 다른 라이브러리와도 비교 해보고 해당 라이브러리만의 장점을 만드는 것이 앞으로의 숙제라고 생각한다.
또한 해당 라이브러리는 통신 결과값 들을 갖게 되기 때문에 메모리 누수의 위험이 있다. 하드웨어 적인 부담도 좀 더 생각하면서 라이브러리를 보완할 필요가 있다.

이번 라이브러리 작성에서 가장 중요시 했던건, `PEP8`에 맞는 코드 컨벤션과 `pythonic`한 코드 작성이었다. 막연히 파이써닉한 코드를 짜야한다라고 생각했던 점들이, 실제로 코드를 작성하면서
필요불가결한 요소로 다가왔고 비록 빠르진 않았지만, 틈틈히 공부했던 내용에 대해서 많이 반여하면서 굉장히 즐겁게 코딩할 수 있었다. 앞으로도 계속 공부하고 좋은 방향으로 버전업 해 볼 예정이다.


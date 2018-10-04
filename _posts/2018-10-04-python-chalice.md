---
title: "Python: Chalice를 이용한 서버리스 이미지 호스팅 제작기"
image:
  path: /images/post-imgs/imager-case-with-sqs.jpg
  thumbnail: /images/post-imgs/imager-case-with-sqs.jpg
categories:
  - Python
  - Chalice
  - Serverless
tags: 
  - chalice
  - serverless
---

### [Chalice](https://chalice.readthedocs.io/en/latest/)를이용한 서버리스 이미지 호스팅 제작기
[Chalic github](https://github.com/aws/chalice)

설계는 이러하다.
![image-center]({{ '/images/post-imgs/imager-case-with-sqs.jpg' | absolute_url }}){: .align-center}

이번 포스트에서는 저기 보이는`SQS -> Lambda -> S3`부분을 다뤄볼까 한다.
Python에는 Zappa나 Chalice처럼 AWS Lambda를 이용한 서버리스 웹 / 프로그램을 구현하는데 큰 도움을 주는 라이브러리가 존재한다.

나는 주로 Flask Web을 Zappa를 통해 서버리스 웹으로 띄우고 하는데, 이번에는 `SQS -> Lambda` 이부분이 현재 Zappa에선 지원이 되지않아, Chalice를 이용해보기로 하였다.

아직 작성중..

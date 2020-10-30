---
layout: post
title: "Python: 파이썬 Chalice를 이용한 서버리스 이미지 호스팅 제작기"
categories:
  - Python
  - Chalice
  - Serverless
tags: 
  - chalice
  - serverless
---

### Chalice를이용한 서버리스 이미지 호스팅 제작기
이번에 이미지를 적재하는 시스템을 만들게되었는데, 안타깝게도 우리 회사에는 서버관리자가 따로 없기 때문에 서버리스로 구현하기로 하였다.
리서치 하다가 결국엔 Chalice라는 라이브러리에 도달하였고, 뜻을 찾아보니 성배라고한다. 이유는 모르겠네?
* [Chalice github](https://github.com/aws/chalice)
* [Chalice docs](https://chalice.readthedocs.io/en/latest/)

설계는 이러하다.
![image-center]({{ '/images/post-imgs/imager-case-with-sqs.jpg' | absolute_url }}){: .align-center}

시스템 아키텍처에서 보이는`SQS -> Lambda -> S3` 부분을Chalice로 구현해보려고 한다. 사실 `S3 -> Lambda -> S3` 같은 설계도 가능한데, 이번에는 SQS에서 JSON형식으로 Message를 받아 처리하고 싶어 이러한 설계를 진행하였다. 참고로 `SQS -> Lambda` 요금은 공짜라는 사실!

AWS Lambda를 이용한 서버리스 웹 / 프로그램을 구현하는데 큰 도움을 주는 라이브러리가 존재하는데, Node.js에서는 Serverless, Python에는 Zappa나 Chalice를 예로 들수 있다.
나는 주로 Flask Web을 Zappa를 통해 서버리스 웹으로 띄우곤 하는데, 이번에는 `SQS -> Lambda`부분이 현재 Zappa에선 지원이 되지않아, Chalice를 이용해보기로 하였다.

1. 상기에서 논한 것 처럼 Chalice는 간편하게 SQS이벤트를 캐치하여 Lambda함수를 실행시키는 것이 가능하다.
Chalice를 이용하기 앞서, AWS Credentials을 등록해야한다.
```bash
$ mkdir ~/.aws
$ cat >> ~/.aws/config
[default]
aws_access_key_id=YOUR_ACCESS_KEY_HERE
aws_secret_access_key=YOUR_SECRET_ACCESS_KEY
region=YOUR_REGION (such as us-west-2, us-west-1, etc)
```

2. 가상환경에 진입 후, Chalice를 설치하자.
```bash
$ pip install chalice
```

3. 확인해보자.
```bash
$ chalice --help
Usage: chalice [OPTIONS] COMMAND [ARGS]...
...
```
4. 잘 설치되었다. 이제 프로젝트를 생성하자.
```bash
$ chalice new-project imager
```
5. 디렉토리로 이동해보자.
```bash
$ cd imager
$ ls -la
drwxr-xr-x   .chalice
-rw-r--r--   app.py
-rw-r--r--   requirements.txt
```
6. app.py을 열어보자.
```bash
from chalice import Chalice
app = Chalice(app_name='imager')
@app.route('/')
def index():
    return {'hello': 'world'}
```
7. 디플로이를 해보자.
```bash
$ chalice deploy
...
Initiating first time deployment...
https://qxea58oupc.execute-api.us-west-2.amazonaws.com/api/
```
8. 확인해보자.
```bash
$ curl https://qxea58oupc.execute-api.us-west-2.amazonaws.com/api/
{"hello": "world"}
```

아주 잘 동작하는걸 확인할 수 있다. AWS Credentials에 큰 문제만 없다면 정상동작할 것이고, Deploy가 안될경우에는, 입력한 AWS key에 해당 <U>Lambda나 APIGateway에 대한 권한</U>이 열려있는지 확인해보자.
{: .notice--warning}

기본적인 세팅이 완료되었다면, 각자 마음대로 요리해보자. 나의 프로젝트 구조는 결과적으로 이러하다.

![image-left]({{ '/images/post-imgs/imager-tree.png' | absolute_url }}){: .align-left}
app.py
: SQS와 맵핑하는 실제 Working 코드가 쓰여있다.

chalicelib
: app.py나 다른 파이썬 코드에 자신이 만든 <U>커스텀 모듈을 import할 경우</U>에는, `chalicelib` 디렉토리를 만들어 넣어줘야 문제없이 deploy된다.

S3_manager.py
: S3 Bucket CRUD 코드이다.

aws.py
: S3_manager.py에서 이용하는 AWS Credentials 정보가 녹아져있다.

imager.py
: 이미지를 압축하는 코드이다.

Zappa의 zappa_settings.json과 같은 configuration 설정은 `.chalice/config.json`에서 가능하다.
{: .notice--info}

현재 config.json은 이렇다. 자세한 내용은 [공식문서](https://chalice.readthedocs.io/en/latest/topics/configfile.html)를 참고해보자.
```bash
$ cat .chalice/config.json
{
  "version": "2.0",
  "app_name": "imager",
  "stages": {
    "dev": {
      "api_gateway_stage": "api",
          "lambda_memory_size": 128
    }
  }
}
```

이제 SQS를 만들어보자. `AWS Console -> SQS -> Create New Queue`를 해주면 된다. Queue에 네임은 imager-queue로 명명하였다.
![image-center]({{ '/images/post-imgs/aws-sqs-imager-queue.png' | absolute_url }}){: .align-center}
자 이제 `SQS -> Lambda -> S3`를 구현하기 위한 코드는 아래와 같다.
```python
# app.py
from chalice import Chalice
from chalicelib.s3_manager import S3ManagerCore
import logging
import json
import time

app = Chalice(app_name='imager')
app.debug = True
app.log.setLevel(logging.DEBUG)

@app.on_sqs_message(queue='imager-queue', batch_size=1)
def build(event):
    #app.log.info("Event: %s", event.to_dict())
    event = event.to_dict()
    status_code = 200
    records = event.get('Records', None)
    try:
        if records is not None and records:
            for rec in records:
                print(rec)
                try:
                    body = json.loads(rec['body'])
                    filename = body.get('filename')
                    url = body.get('url')
                    build_core(filename, url)
                except Exception as e:
                    app.log.error(e)
    except Exception as e:
        status_code = 400

    response = {
        "statusCode": status_code,
        "body": json.dumps(event)
    }
    return response

# S3에 넣는 로직
def build_core(filename, url):
    S3ManagerCore.build(filename, url)
```
`@app.on_sqs_message(queue='imager-queue', batch_size=1)` 이라고 데코레이터를 먹이면 다음과 같이 event를 받아 올 수 있다. 이렇게 편할 수 없다. 또한, batch_size로 한번에 가져올 수 있는 Queue의 갯수를 정할 수 있다.

Deploy를 해보겠다.
```bash
$ chalice deploy
Creating deployment package.

Could not install dependencies:
pillow==5.2.0
You will have to build these yourself and vendor them in
the chalice vendor folder.

Your deployment will continue but may not work correctly
if missing dependencies are not present. For more information:
http://chalice.readthedocs.io/en/latest/topics/packaging.html

Creating IAM role: imager-dev
Creating lambda function: imager-dev-build
Subscribing imager-dev-build to SQS queue imager-queue
Resources deployed:
  - Lambda ARN: arn:aws:lambda:ap-northeast-2:xxxxxxxx:function:imager-dev-build
```
문제없이 업로드 되었다. IAM role도 자동으로 생성되어 편리하다. 콘솔에 들어가 확인해보면, 정상적으로 SQS와 맵핑이 된것을 확인할수 있다.

![image-center]({{ '/images/post-imgs/aws-lambda-sqs.png' | absolute_url }}){: .align-center}

실제로 SQS에 Queue를 날려보니 S3에 문제없이 적재되는 것을 확인하였다. 꼭 다음과 같은 방식이 아니더라도 여러 방면으로 활용성이 좋아보이는 Chalice였다. Zappa와는 다른점으로는 `app.py`에서 정의한 function 만큼 각기 별도의 Lambda로 Deploy되어 여러개의 Lambda를 하나의 소스에서 통합관리가 가능해 보였다. (Zappa 같은경우, 하나의 Lambda에 url path에 따른 분기)

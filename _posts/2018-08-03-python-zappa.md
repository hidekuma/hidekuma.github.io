---
title: "Python: 파이썬 Zappa로 AWS Lambda에 서버리스 Web 구현하기"
excerpt_separator: <!--more-->
categories:
  - Serverless
  - AWS
  - Lambda
  - APIGateway
  - Zappa
tags: 
  - serverless
  - aws
  - lambda
  - apigateway
  - zappa
---

## Flask + Zappa > AWS Lambda
`Flask`나 `django`와 같은 파이썬 웹을 서버리스로 띄우는 데 `Zappa`는 매우 용이하다. [`Chalice`](https://hidekuma.github.io/python/chalice/serverless/python-chalice/)와 달리 디플로이 했을 시, 하나의 `Lambda함수`에서 모든 `Requests`를 관리해준다. 또한 `Flask Web`에서 하듯이 코드를 자유롭게 짜면 되기 때문에, 해당 웹 프레임워크에 친숙한 사람이라면, `Zappa`를 채택하는 것이 옳다.
* [Zappa github](https://github.com/Miserlou/Zappa)

1. 가상환경에 진입한다. 나의 경우 `(zappa-test)`라고 가상환경을 만들었다. <br>[**가상환경을 만드는 방법**에 대해 알고 싶다면, 해당 포스트를 참고하면 된다.](https://hidekuma.github.io/python/virtualenv/virtualenvwrapper/python-virtualenv-wrapper/)

<!--more-->
2. `zappa` 설치.
```bash
(zappa-test) $ pip install zappa
```

3. `zappa init` 한다.
```bash
(zappa-test) $ zappa init
```
`zappa init` 커맨드를 치면, 몇 가지 질의를 해온다. 
- What do you want to call this environment (default 'dev')
	: `APIGateway -> stage`와 연결된다. 자신이 현재 만드는 환경을 정의해주면된다. Deploy 용도라면 `production`등으로 정의해주면 된다. 나중에 변경가능하니, 지금은 `dev`로 정의하자. 
- What do you want to call your bucket? (default 'zappa-xxxxxx')
	: 이미 Lambda function 소스를 `zip`형태로 적재할 `S3 Bucket`이 있다면 넣어주고, 없다면 네이밍 해주면 해당 버킷이 S3에 생성된다.
- Would you like to deploy this application globally? (default 'n') [y/n/(p)rimary]
	: 글로벌 하게 사용할 것인지를 묻는데, 각 리전마다 람다를 사용해야 할 경우에만, `y`로 하자. 혹시 `y`로 한다면, `zappa_settings.json`에 리전별 `config`가 생성이 될것이다.
- Where is your app's function?
	: 실행 시킬 혹은 실행 시킬 예정인 py파일을 넣어주면 된다. 나같은 경우 `app.py`으로 파일을 만들 예정이니 `app.app`이라고 넣었다.
4. `zappa_settings.json` 체크.
```bash
(zappa-test) $ tree
.
└── zappa_settings.json
0 directories, 1 file
```
```bash
(zappa-test) $ cat zappa_settings.json
{
    "dev": {
        "app_function": "app.app",
        "aws_region": "ap-northeast-2",
        "profile_name": "default",
        "project_name": "zappa-test",
        "runtime": "python3.6",
        "s3_bucket": "zappa-test"
    }
}
```

5. Web Framework은 `Flask`를 사용한다. (django도 지원함.)
```python
#app.py
import sys
from flask import Flask, redirect, request, jsonify, url_for, render_template
app = Flask(__name__, static_url_path='')
@app.route('/', methods=['GET'])
def index():
		return 'hello zappa!'
if __name__ == '__main__':
    if len(sys.argv) > 1:
        app.debug = True
        app.jinja_env.auto_reload = True
        app.config['TEMPLATES_AUTO_RELOAD'] = True
        app.run(host='0.0.0.0', port=4000)
    else:
        app.run(host='0.0.0.0')
```
개발환경을 구분하기 위해 러닝 커맨드에 인자를 받을 수 있게끔 구성하였다. 
- app.debug
	: 오류사항이 있을 시, view에 오류내용이 그려진다.
- app.jinja_env.auto_reload / app.config['TEMPLATES_AUTO_RELOAD'] 
	: `jinja / html / py` 파일 변경점이 있을 시, 바로 재로드한다.

6. `Flask` 설치.
```bash
(zappa-test) $ pip install flask
```

7. 실행시켜보자.
```bash
(zappa-test) $ python app.py test
 * Serving Flask app "app" (lazy loading)
 * Environment: production
   WARNING: Do not use the development server in a production environment.
   Use a production WSGI server instead.
 * Debug mode: on
 * Running on http://0.0.0.0:4000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 954-150-263
```
![image-center]({{ '/images/post-imgs/hello-zappa.png' | absolute_url }}){: .align-center}
잘 동작하는 걸 확인했다.

8. `AWS Lambda`에 업로드해보자.
[AWS credentials file](https://aws.amazon.com/ko/blogs/security/a-new-and-standardized-way-to-manage-credentials-in-the-aws-sdks/) 설정은 이미 완료되어있어야한다. 잘 모르겠다면 문서를 참고하자. 
```bash
(zappa-test) $ zappa deploy dev
Calling deploy for stage dev..
Warning! Your project and virtualenv have the same name! You may want to re-create your venv with a new name, or explicitly define a 'project_name', as this may cause errors.
Downloading and installing dependencies..
 - sqlite==python36: Using precompiled lambda package
Packaging project as zip.
Uploading zappa-test-dev-1538977358.zip (5.7MiB)..
100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████| 5.95M/5.95M [00:00<00:00, 8.63MB/s]
Scheduling..
Scheduled zappa-test-dev-zappa-keep-warm-handler.keep_warm_callback with expression rate(4 minutes)!
Uploading zappa-test-dev-template-1538977365.json (1.6KiB)..
100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1.64K/1.64K [00:00<00:00, 47.1KB/s]
Waiting for stack zappa-test-dev to create (this can take a bit)..
100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 4/4 [00:09<00:00,  2.66s/res]
Deploying API Gateway..
Deployment complete!: https://scv5bcbvag.execute-api.ap-northeast-2.amazonaws.com/dev
```
![image-center]({{ '/images/post-imgs/zappa-lambda-output.png' | absolute_url }}){: .align-center}
잘 업로드 되었다.

9. `undeploy`를 해보자
```bash
(zappa-test) $ zappa undeploy dev
Calling undeploy for stage dev..
Are you sure you want to undeploy? [y/n] y
Deleting API Gateway..
Waiting for stack zappa-test-dev to be deleted..
Unscheduling..
Unscheduled zappa-test-dev-zappa-keep-warm-handler.keep_warm_callback.
Deleting Lambda function..
Done!
```
깨끗하게 삭제하고 싶다면, 해당 커맨드로는 `AWS IAM Role`과 `S3 Bucket`은 삭제되지 않으니, 콘솔에 접속하여 직접 지워주면 된다.

**Deploy시 주의할 점!** <br> `*.pyc` 캐시 파일이 같이 업로드 되면, `TypeError: 'NoneType' object is not callable`에러가 발생한다. 따라서 프로젝트 내 파이썬 캐시파일을 모두 지워주는 편이 좋다. 예를 들면, `find . -name \*.pyc -delete` 해당 쉘로 손쉽게 지울수 있다.
{: .notice--warning}


---
## 자주 쓰는 Commands
- zappa deploy [stage]
: 해당 스테이지로 소스를 업로드한다.

- zappa update [stage]
: 해당 스테이지를 업데이트한다.

- zappa undeploy [stage]
: 해당 스테이지를 삭제한다.

- zappa tail [stage]
: 해당 스테이지의 로그를 볼 수 있다. `ex) zappa tail dev --since 1m --filter 'error'`

- zappa rollback [stage] -n [숫자]
: 숫자만큼 디플로이 했던 횟수를 취소한다. 즉 현 시점에서 [숫자]회 전 디플로이 환경으로 롤백시켜준다. `ex) zappa rollback dev -n 1 (1회 전 디플로이 환경으로 롤백)`


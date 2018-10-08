---
title: "Python: Zappa로 AWS Lambda에 서버리스 Web 구현하기"
excerpt_separator: <!--more-->
image:
  path: /images/post-imgs/zappa-banner.jpeg
  thumbnail: /images/post-imgs/zappa-banner.jpeg
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
Collecting zappa
Collecting base58==1.0.0 (from zappa)
Collecting python-slugify==1.2.4 (from zappa)
  Using cached https://files.pythonhosted.org/packages/9f/77/ab7134b731d0e831cf82861c1ab0bb318e80c41155fa9da18958f9d96057/python_slugify-1.2.4-py2.py3-none-any.whl
Collecting six>=1.11.0 (from zappa)
  Using cached https://files.pythonhosted.org/packages/67/4b/141a581104b1f6397bfa78ac9d43d8ad29a7ca43ea90a2d863fe3056e86a/six-1.11.0-py2.py3-none-any.whl
Collecting PyYAML==3.12 (from zappa)
Collecting tqdm==4.19.1 (from zappa)
  Using cached https://files.pythonhosted.org/packages/c0/d3/7f930cbfcafae3836be39dd3ed9b77e5bb177bdcf587a80b6cd1c7b85e74/tqdm-4.19.1-py2.py3-none-any.whl
Collecting requests>=2.10.0 (from zappa)
  Using cached https://files.pythonhosted.org/packages/65/47/7e02164a2a3db50ed6d8a6ab1d6d60b69c4c3fdf57a284257925dfc12bda/requests-2.19.1-py2.py3-none-any.whl
Collecting hjson==3.0.1 (from zappa)
Collecting wsgi-request-logger==0.4.6 (from zappa)
Collecting botocore>=1.7.19 (from zappa)
  Downloading https://files.pythonhosted.org/packages/38/8e/014195a281c82b952e026f7d741d558fab18fd1404becb72407e165c614e/botocore-1.12.18-py2.py3-none-any.whl (4.7MB)
    100% |████████████████████████████████| 4.7MB 3.2MB/s
Collecting lambda-packages==0.20.0 (from zappa)
Collecting jmespath==0.9.3 (from zappa)
  Using cached https://files.pythonhosted.org/packages/b7/31/05c8d001f7f87f0f07289a5fc0fc3832e9a57f2dbd4d3b0fee70e0d51365/jmespath-0.9.3-py2.py3-none-any.whl
Collecting python-dateutil<2.7.0,>=2.6.1 (from zappa)
  Using cached https://files.pythonhosted.org/packages/4b/0d/7ed381ab4fe80b8ebf34411d14f253e1cf3e56e2820ffa1d8844b23859a2/python_dateutil-2.6.1-py2.py3-none-any.whl
Collecting kappa==0.6.0 (from zappa)
Collecting argcomplete==1.9.3 (from zappa)
  Using cached https://files.pythonhosted.org/packages/0d/f2/058910b2c732092175875820177dae9d390e71a5f30a9895f92e6a6ca466/argcomplete-1.9.3-py2.py3-none-any.whl
Collecting toml>=0.9.4 (from zappa)
  Downloading https://files.pythonhosted.org/packages/a2/12/ced7105d2de62fa7c8fb5fce92cc4ce66b57c95fb875e9318dba7f8c5db0/toml-0.10.0-py2.py3-none-any.whl
Collecting boto3>=1.4.7 (from zappa)
  Downloading https://files.pythonhosted.org/packages/1e/ab/9694da3ac1ee3e7f8a8c7d45d8135dad49dd01bdb67e8951c628a8ccc20d/boto3-1.9.18-py2.py3-none-any.whl (128kB)
    100% |████████████████████████████████| 133kB 10.7MB/s
Collecting troposphere>=1.9.0 (from zappa)
Collecting durationpy==0.5 (from zappa)
Collecting pip<=10.1.0,>=9.0.1 (from zappa)
  Using cached https://files.pythonhosted.org/packages/0f/74/ecd13431bcc456ed390b44c8a6e917c1820365cbebcb6a8974d1cd045ab4/pip-10.0.1-py2.py3-none-any.whl
Collecting Werkzeug>=0.14 (from zappa)
  Using cached https://files.pythonhosted.org/packages/20/c4/12e3e56473e52375aa29c4764e70d1b8f3efa6682bef8d0aae04fe335243/Werkzeug-0.14.1-py2.py3-none-any.whl
Collecting docutils>=0.12 (from zappa)
  Using cached https://files.pythonhosted.org/packages/36/fa/08e9e6e0e3cbd1d362c3bbee8d01d0aedb2155c4ac112b19ef3cae8eed8d/docutils-0.14-py3-none-any.whl
Requirement already satisfied: wheel>=0.30.0 in /Users/hideo/.virtualenvs/zappa-test/lib/python3.6/site-packages (from zappa) (0.32.1)
Collecting future==0.16.0 (from zappa)
Collecting Unidecode>=0.04.16 (from python-slugify==1.2.4->zappa)
  Using cached https://files.pythonhosted.org/packages/59/ef/67085e30e8bbcdd76e2f0a4ad8151c13a2c5bce77c85f8cad6e1f16fb141/Unidecode-1.0.22-py2.py3-none-any.whl
Collecting chardet<3.1.0,>=3.0.2 (from requests>=2.10.0->zappa)
  Using cached https://files.pythonhosted.org/packages/bc/a9/01ffebfb562e4274b6487b4bb1ddec7ca55ec7510b22e4c51f14098443b8/chardet-3.0.4-py2.py3-none-any.whl
Collecting certifi>=2017.4.17 (from requests>=2.10.0->zappa)
  Using cached https://files.pythonhosted.org/packages/df/f7/04fee6ac349e915b82171f8e23cee63644d83663b34c539f7a09aed18f9e/certifi-2018.8.24-py2.py3-none-any.whl
Collecting idna<2.8,>=2.5 (from requests>=2.10.0->zappa)
  Using cached https://files.pythonhosted.org/packages/4b/2a/0276479a4b3caeb8a8c1af2f8e4355746a97fab05a372e4a2c6a6b876165/idna-2.7-py2.py3-none-any.whl
Collecting urllib3<1.24,>=1.21.1 (from requests>=2.10.0->zappa)
  Using cached https://files.pythonhosted.org/packages/bd/c9/6fdd990019071a4a32a5e7cb78a1d92c53851ef4f56f62a3486e6a7d8ffb/urllib3-1.23-py2.py3-none-any.whl
Collecting click>=5.1 (from kappa==0.6.0->zappa)
  Using cached https://files.pythonhosted.org/packages/fa/37/45185cb5abbc30d7257104c434fe0b07e5a195a6847506c074527aa599ec/Click-7.0-py2.py3-none-any.whl
Collecting placebo>=0.8.1 (from kappa==0.6.0->zappa)
Collecting s3transfer<0.2.0,>=0.1.10 (from boto3>=1.4.7->zappa)
  Using cached https://files.pythonhosted.org/packages/d7/14/2a0004d487464d120c9fb85313a75cd3d71a7506955be458eebfe19a6b1d/s3transfer-0.1.13-py2.py3-none-any.whl
Collecting cfn-flip>=1.0.2 (from troposphere>=1.9.0->zappa)
Installing collected packages: base58, Unidecode, python-slugify, six, PyYAML, tqdm, chardet, certifi, idna, urllib3, requests, hjson, wsgi-request-logger, jmespath, docutils, python-dateutil, botocore, lambda-packages, click, placebo, s3transfer, boto3, kappa, argcomplete, toml, cfn-flip, troposphere, durationpy, pip, Werkzeug, future, zappa
  Found existing installation: pip 18.1
    Uninstalling pip-18.1:
      Successfully uninstalled pip-18.1
Successfully installed PyYAML-3.12 Unidecode-1.0.22 Werkzeug-0.14.1 argcomplete-1.9.3 base58-1.0.0 boto3-1.9.18 botocore-1.12.18 certifi-2018.8.24 cfn-flip-1.0.3 chardet-3.0.4 click-7.0 docutils-0.14 durationpy-0.5 future-0.16.0 hjson-3.0.1 idna-2.7 jmespath-0.9.3 kappa-0.6.0 lambda-packages-0.20.0 pip-10.0.1 placebo-0.8.2 python-dateutil-2.6.1 python-slugify-1.2.4 requests-2.19.1 s3transfer-0.1.13 six-1.11.0 toml-0.10.0 tqdm-4.19.1 troposphere-2.3.3 urllib3-1.23 wsgi-request-logger-0.4.6 zappa-0.46.2
```

3. `zappa init` 한다.
```bash
(zappa-test) $ zappa init
███████╗ █████╗ ██████╗ ██████╗  █████╗
╚══███╔╝██╔══██╗██╔══██╗██╔══██╗██╔══██╗
  ███╔╝ ███████║██████╔╝██████╔╝███████║
 ███╔╝  ██╔══██║██╔═══╝ ██╔═══╝ ██╔══██║
███████╗██║  ██║██║     ██║     ██║  ██║
╚══════╝╚═╝  ╚═╝╚═╝     ╚═╝     ╚═╝  ╚═╝
Welcome to Zappa!
\
Zappa is a system for running server-less Python web applications on AWS Lambda and AWS API Gateway.
This `init` command will help you create and configure your new Zappa deployment.
Let's get started!
Your Zappa configuration can support multiple production stages, like 'dev', 'staging', and 'production'.
\
What do you want to call this environment (default 'dev'): dev
AWS Lambda and API Gateway are only available in certain regions. Let's check to make sure you have a profile set up in one that will work.
Okay, using profile default!
Your Zappa deployments will need to be uploaded to a private S3 bucket.
If you don't have a bucket yet, we'll create one for you too.
\
What do you want to call your bucket? (default 'zappa-2nbfdlibr'): zappa-test
\
What's the modular path to your app's function?
This will likely be something like 'your_module.app'.
\
Where is your app's function?: app.app
You can optionally deploy to all available regions in order to provide fast global service.
If you are using Zappa for the first time, you probably don't want to do this!
\
Would you like to deploy this application globally? (default 'n') [y/n/(p)rimary]: n
Okay, here's your zappa_settings.json:
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
\
Does this look okay? (default 'y') [y/n]: y
\
Done! Now you can deploy your Zappa application by executing:
        $ zappa deploy dev
After that, you can update your application code with:
        $ zappa update dev
\
To learn more, check out our project page on GitHub here: https://github.com/Miserlou/Zappa
and stop by our Slack channel here: https://slack.zappa.io
Enjoy!,
 ~ Team Zappa!
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
\
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
\
app = Flask(__name__, static_url_path='')
\
@app.route('/', methods=['GET'])
def index():
		return 'hello zappa!'
\
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
Collecting flask
  Using cached https://files.pythonhosted.org/packages/7f/e7/08578774ed4536d3242b14dacb4696386634607af824ea997202cd0edb4b/Flask-1.0.2-py2.py3-none-any.whl
Requirement already satisfied: Werkzeug>=0.14 in /Users/hideo/.virtualenvs/zappa-test/lib/python3.6/site-packages (from flask) (0.14.1)
Requirement already satisfied: click>=5.1 in /Users/hideo/.virtualenvs/zappa-test/lib/python3.6/site-packages (from flask) (7.0)
Collecting itsdangerous>=0.24 (from flask)
Collecting Jinja2>=2.10 (from flask)
  Using cached https://files.pythonhosted.org/packages/7f/ff/ae64bacdfc95f27a016a7bed8e8686763ba4d277a78ca76f32659220a731/Jinja2-2.10-py2.py3-none-any.whl
Collecting MarkupSafe>=0.23 (from Jinja2>=2.10->flask)
Installing collected packages: itsdangerous, MarkupSafe, Jinja2, flask
Successfully installed Jinja2-2.10 MarkupSafe-1.0 flask-1.0.2 itsdangerous-0.24
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


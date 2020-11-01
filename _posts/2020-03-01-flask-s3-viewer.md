---
layout: post
title: "Python: 파이썬으로 S3 데이터 브라우징하기"
excerpt_separator: <!--more-->
categories:
  - python
  - aws
tags:
  - flask-s3-viewer
---

# 오픈소스, Flask S3 Viewer
<iframe style="width: 100%; max-width:640px; height:480px;" src="https://youtube.com/embed/tu8U6UR9BPA?start=0" allowfullscreen="" frameborder="0" ></iframe>

## 설치
```bash
pip install flask flask_s3_viewer
```

## 세팅
```python
from flask import Flask

from flask_s3_viewer import FlaskS3Viewer
from flask_s3_viewer.aws.ref import Region

# Init Flask
app = Flask(__name__)

# Init Flask S3Viewer
s3viewer = FlaskS3Viewer(
    # Flask App
    app,
    template_namespace='mdl',
    # Namespace must be unique
    namespace='flask-s3-viewer',
    # Hostname, e.g. Cloudfront endpoint
    object_hostname='http://flask-s3-viewer.com',
    # Put your AWS's profile name and Bucket name
    config={
        'profile_name': 'PROFILE_NAME',
        'bucket_name': 'S3_BUCKET_NAME'
    }
)

# Register Flask S3Viewer's router
s3viewer.register()

if __name__ == '__main__':
    app.run(debug=True, port=3000)
```
## 플라스크 실행
```bash
python app.py
```
http://localhost:3000/flask-s3-viewer/files 으로 방문하면, 원하는 화면을 볼 수 있다.
만약, namespace가 변경된다면 접속 url도 변경된다 (http://localhost:3000/{namespace}/files)

자세한 내용은 [docs](https://flask-s3-viewer.readthedocs.io/en/latest/index.html)를 참고하면 빠르게 적용할 수 있다.

---
layout: post
title: "GCP: Firebase를 통한 Serverless 웹사이트 운영후기"
categories:
  - gcp 
  - firebase
  - website
  - serverless
tags:
  - gcp
  - firebase
  - website
  - serverless
---

### Firebase 기반 Serverless 웹 앱 제작
개인적으로 AWS가 더 친숙하고 사용경험이 많기에 사실 해당 프로젝트를 시작할 때, AWS를 사용하는 편이 속도는 나왔겠지만 뭐든 여유로울 때 학습하는 것이 좋기에, 굳이 GCP를 택했다. 또한 1년 Free Trial 300달러는 이것 저것 해보기에 딱 좋았다.

#### 서비스 소개
전세 계약 시에, 간편하게 정보를 조회하고 계약하기 안전한 집인지 소정의 지표를 제공하는 웹 사이트이다. 해당 프로젝트는 2021년 04월 01일을 기준으로 서비스를 종료하므로, 따로 도메인을 남기진 않겠다.
<iframe style="max-width: 320px;" width="320" height="240" src="https://youtube.com/embed/IlFvkp22sqw?start=0" allowfullscreen="" frameborder="0" ></iframe>

----

### 기술스택
서비스 모델 자체가 별도의 큰 데이터베이스나 비즈니스 로직이 필요 없는 룰베이스 기반 시스템이였기에 인프라 구성은 매우 간단하다.

#### Server
Python Flask 기반의 웹 으로, 웹 서버와 API서버로 분리되어있다.
Serverless로 구성하기 위해서 아래와 같이 Cloud Run을 이용하였다. 

![image-center]({{ '/images/post-imgs/iskangtong/apiandweb.png' | absolute_url }}){: .align-center}

개인 프로젝트였기에 협업 가능성은 없었지만, API 제작시에 필수적으로 문서화를 진행하는 편이므로, 아래와 같이 Swagger를 통해 API Documentation을 자동화했다.

![image-center]({{ '/images/post-imgs/iskangtong/swagger.png' | absolute_url }}){: .align-center}

#### Deploy
Cloud Build를 통해 Python Flask 앱을 빌드한 도커 이미지(Dockerfile)를 만들고 Cloud Run에 배포했다.

```bash
# 도커 이미지를 빌드
gcloud builds submit --tag gcr.io/iskangtong/apiv1

# 도커 이미지를 Cloud Run에 배포하여 서버리스로 운영
gcloud run deploy apiv1 --region asia-northeast1 --platform managed --image gcr.io/iskangtong/apiv1
```

```bash
# Static한 파일을 다루는 Hosting영역을 배포
./node_modules/.bin/firebase deploy --only hosting  
```

#### Routing
Firebase를 통해 호스팅되는 트래픽을 Cloud Run에 배포한 서버리스 웹으로 전달해야하는데, 이 설정은 firebase.json을 통해 진행하면 된다.
```bash
{
  "hosting": {
    "public": "static",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ],
    "headers": [
        {
            "source": "/api/@(v1)/**",
            "headers": [ {
                 "key": "Cache-Control",
                 "value": "private, max-age=1800, s-maxage=3600"
            } ]
        },
        {
            "source": "/swaggerui/**",
            "headers": [ {
                 "key": "Cache-Control",
                 "value": "public, max-age=43200, s-maxage=86400"
            } ]
        }
    ],
    "rewrites": [
      {
        "source": "!/@(api|swaggerui)/**",
        "run": {
            "serviceId": "flask",
            "region": "asia-northeast1"
        }
      },
      {
        "source": "/api/v1/**",
        "run": {
            "serviceId": "apiv1",
            "region": "asia-northeast1"
        }
      },
      {
        "source": "/swaggerui/**",
        "run": {
            "serviceId": "apiv1",
            "region": "asia-northeast1"
        }
      }
    ]
  },
  "firestore": {
    "rules": "firestore.rules",
    "indexes": "firestore.indexes.json"
  },
  "storage": {
    "rules": "storage.rules"
  }
}
```

#### Domain
Firebase의 좋은 점은 *.web.app 이라는 도메인을 기본적으로 제공해준다. 따라서, 도메인을 크게 신경쓰지 않는다면 기본으로 제공하는 해당 도메인을 이용했어도 무방했다.

![image-center]({{ '/images/post-imgs/iskangtong/domain.png' | absolute_url }}){: .align-center}

#### Database
Google Firestore를 이용하였고, 인터페이스는 다르지만, Collection과 Document가 개념이 존재하고, NoSQL 데이터베이스와 유사하게 사용가능하였다.

![image-center]({{ '/images/post-imgs/iskangtong/firestore.png' | absolute_url }}){: .align-center}


#### GCP APIs
지난 30일 기준이며, 전체적으로 아래와 같이 GCP API가 사용되었다.

![image-center]({{ '/images/post-imgs/iskangtong/gcpapis.png' | absolute_url }}){: .align-center}

----

### 개발 후기
#### Firebase 사용후기
모든 기능을 사용해보지 않았지만, AWS amplify와 비슷하다고 볼 수있겠다. firebase SDK가 존재하고 사용방법 또한 각 문서를 보면 누구라도 개발할 수 있게끔 되어있었기 때문에 전체적인 개발경험은 매우 좋았다.

몇 가지 커맨드로 빌드와 배포를 간편하게 진행할 수 있었으며, 개인적으로는 Backend 개발자가 하는 일이 없어졌다고 느낄 정도로 편리했다. Firebase 콘솔 페이지도 UI/UX가 매우 간단하며 진입장벽이 매우 낮다고 판단되며 소규모 프로젝트에서 도입해보는 것을 추천한다.

#### 서비스 후기
위에서 말했듯이 서비스 목적이 전세 계약 시에, 계약하기 안전한 집인지 소정의 지표 제공하는 웹이다.

하지만 2021년, 새해가 밝은 지금도 전세가격이 하늘로 치솟고 있어서, 특히 서울에는 여러가지 이유로 안전하지 않은 집이여도 들어갈 수밖에 없는 상황이 되었다.
이게 서비스 종료의 가장 큰 이유이라고 볼 수 있겠다. 전세 매물도 없을 뿐더러 현실은 이것저것 따질 정도로 여유롭지 않기 때문이다.

결코 많은 사용자가 있었다고 말할 수도 없지만, 서비스를 운영하면서 여러번의 이메일을 받았다.
내용에는 건의사항도 있었지만, 감사의 인사도 있었기에 누군가에게는 도움이 되었다고 생각하면 매우 만족스럽다.

간단한 웹 서비스였지만, 공공데이터 포털에서 사용하는 API는 총 16개였다.
참고로 공공데이터 포탈의 API들은 언제 장애가 나도 이상하지 않으므로, 별도로 적재해서 사용하는 것이 바람직하다.

##### 사용된 공공데이터 포탈 API 항목
- [승인] 국토교통부_건축물대장정보 서비스
- [승인] 공동주택가격정보서비스
- [승인] 개별주택가격정보서비스
- [승인] 상업업무용 부동산 매매 신고 자료
- [승인] 연립다세대 전월세 자료
- [승인] 아파트 전월세 자료
- [승인] 단독/다가구 전월세 자료
- [승인] 오피스텔 전월세 신고 조회 서비스
- [승인] 오피스텔 매매 신고 조회 서비스
- [승인] 단독/다가구 매매 실거래 자료
- [승인] 아파트매매 실거래자료
- [승인] 연립다세대 매매 실거래자료
- [승인] 건축물대장정보 서비스
- [승인] 건축물연령정보서비스
- [승인] 토지특성정보서비스
- [승인] 토지소유정보서비스

##### 국가별 사용자
![image-center]({{ '/images/post-imgs/iskangtong/countries.png' | absolute_url }}){: .align-center}

##### 인입 디바이스 비율 및 페이지별 조회수
![image-center]({{ '/images/post-imgs/iskangtong/devices.png' | absolute_url }}){: .align-center}
![image-center]({{ '/images/post-imgs/iskangtong/pages.png' | absolute_url }}){: .align-center}

##### 유니크 사용자 추이
![image-center]({{ '/images/post-imgs/iskangtong/users.png' | absolute_url }}){: .align-center}

---
title: "AWS DynamoDB + API Gateway: API Gateway를 사용해 DynamoDB CRUD API만들기"
excerpt_separator: <!--more-->
categories:
  - DynamoDB
  - API Gateway
  - AWS
tags: 
  - aws 
  - dynamo
---

## DynamoDB

1. 먼저 DynamoDB에 Table을 만들어준다.
![image-left]({{ '/images/post-imgs/dynamo-with-api/dynamo-1.png' | absolute_url }}){: .align-center}

2. Create item으로 테스트 용 데이터를 넣어주자.
![image-center]({{ '/images/post-imgs/dynamo-with-api/dynamo-5.png' | absolute_url }}){: .align-center}
<!--more-->

3. API Gateway를 설정하기 전에 Role을 만들어주어야 한다.
![image-center]({{ '/images/post-imgs/dynamo-with-api/role-1.png' | absolute_url }}){: .align-center}
그림과 같이 <U>DynamoDB에 Full Access</U>를 할 수있는 `Policy`를 가지는 `Role`을 만들어준다.

4. 이제 API Gateway를 만들어준다.
![image-center]({{ '/images/post-imgs/dynamo-with-api/api-1.png' | absolute_url }}){: .align-center}
현재는 테스트로 해보는 것 이기때문에 엣지 옵티마이징은 사용하지 않고 리저널로 엔드포인트 타입을 설정한다.

5. API가 만들어지면, 리소스를 생성한다.
![image-center]({{ '/images/post-imgs/dynamo-with-api/api-2.png' | absolute_url }}){: .align-center}
RESTful한 API 규칙을 따라줘야 추후에 API Gateway 캐싱을 이용할 수 있다.
리소스 네임과 리소 패스를 `{test_id}`로 설정하여 DynamoDB의 PK를 받을 수 있도록 만들자.

6. 리소스가 만들어지면, 해당 리소스에 `GET` 메소드를 생성한다.
![image-center]({{ '/images/post-imgs/dynamo-with-api/api-3.png' | absolute_url }}){: .align-center}
* Intergration type: `AWS Service`
* AWS Region: `ap-northeast-2`
* HTTP method: `POST` (추후 생성하는 모든 메소드에서도 POST로 통일)
* Action: `GetItem`
* Execution role: 3번에서 만든 `Role의 ARN`을 넣어준다.

.................ing



![image-center]({{ '/images/post-imgs/dynamo-with-api/api-4.png' | absolute_url }}){: .align-center}
![image-center]({{ '/images/post-imgs/dynamo-with-api/api-5.png' | absolute_url }}){: .align-center}
![image-center]({{ '/images/post-imgs/dynamo-with-api/api-6.png' | absolute_url }}){: .align-center}

---
title: "AWS: DynamoDB + API Gateway를 사용해 Serverless RESTful API 만들기"
excerpt_separator: <!--more-->
categories:
  - DynamoDB
  - API Gateway
  - AWS
  - Serverless
tags: 
  - aws 
  - dynamo
  - serverless
---

# Serverless API/Database
AWS Lambda를 이용하지 않고, 서버리스 API/Database를 만드는 법을 공유한다.


## DynamoDB + API Gateway

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
현재는 테스트기 때문에 `Edge optimized`는 사용하지 않고 `Regional`로  `Endpoint Type`을 설정한다.

5. API가 만들어지면, 리소스를 생성한다.
![image-center]({{ '/images/post-imgs/dynamo-with-api/api-2.png' | absolute_url }}){: .align-center}
`RESTful`한 API 규칙을 따라줘야 추후에 `API Gateway caching`을 이용할 수 있다. (paramter까지 캐싱하지 않기 때문에)
`Resource name/path`를 `{test_id}`로 설정하여 `DynamoDB`의 `PK`를 받을 수 있도록 만들자.

6. 리소스가 만들어지면, 해당 리소스에 `GET` 메소드를 생성한다.
![image-center]({{ '/images/post-imgs/dynamo-with-api/api-3.png' | absolute_url }}){: .align-center}
* Intergration type: `AWS Service`
* AWS Region: `ap-northeast-2`
* AWS Service: `DynamoDB`
* HTTP method: `POST`
  - `API Gateway` > `DynamoDB`의 통합 호출은 모두 `POST`로 진행해줘야 정상동작 하는 것 같다. 추후에 다시 확인해 볼 예정.
* Action: `GetItem`
* Execution role: 3번에서 만든 `Role의 ARN`을 넣어준다.

7. `API Gateway`의 해당 method의  `Intergration Request`를 클릭하면, 그림과 같이 가장 하단에 `Mapping Templates`가 있다.
![image-center]({{ '/images/post-imgs/dynamo-with-api/api-4.png' | absolute_url }}){: .align-center}
* 템플릿은 공식문서를 참고하면 된다.
* `application/json`을 추가하고 `test_id`를 이용해서 `test` 테이블을 조회하는 탬플릿을 작성하고 save한다.
```json
{ 
    "TableName": "test",
    "Key": {
      "test_id": { 
        "S": "$input.params('test_id')"
      }
    }
}
```

8. 뒤로가기 한 후에 가장 왼편을 보면 테스트라고 있다. 해당 부분을 클릭하면 다음과 같은 화면을 확인할 수 있다.
![image-center]({{ '/images/post-imgs/dynamo-with-api/api-5.png' | absolute_url }}){: .align-center}
우리가 `DB`에 <U>테스트</U>라는 `test_id`를 가지는 데이터를 넣어놨기 때문에, 테스트라는 값을 `Path`에 넣고 실행해보면, 정상적으로 리턴이 오는 걸 확인할 수 있다.

9. `GET`은 완성하였다. 이제 나머지 메소드들도 만들어보자.  
![image-center]({{ '/images/post-imgs/dynamo-with-api/api-6.png' | absolute_url }}){: .align-center}
흐름은 다 똑같으므로, 하단에는 리소스 할당 내용과 템플릿만 공유하겠다.

---

## GET
* Intergration type: `AWS Service`
* AWS Region: `ap-northeast-2`
* AWS Service: `DynamoDB`
* HTTP method: `POST`
* Action: `GetItem`
* Execution role: 3번에서 만든 `Role의 ARN`
* Intergration Request - Mapping Templates - application/json
```json
{ 
    "TableName": "test",
    "Key": {
      "test_id": { 
        "S": "$input.params('test_id')"
      }
    }
}
```

## POST
* Intergration type: `AWS Service`
* AWS Region: `ap-northeast-2`
* AWS Service: `DynamoDB`
* HTTP method: `POST`
* Action: `PutItem`
* Execution role: 3번에서 만든 `Role의 ARN`
* Intergration Request - Mapping Templates - application/json
```json
{ 
    "TableName": "test",
    "Item": {
          "test_id": {
              "S": "$input.path('$.test_id')"
          },
          "value": {
              "S": "$input.path('$.value')"
          }
      }
}
```

## PUT
* Intergration type: `AWS Service`
* AWS Region: `ap-northeast-2`
* AWS Service: `DynamoDB`
* HTTP method: `POST`
* Action: `UpdateItem`
* Execution role: 3번에서 만든 `Role의 ARN`
* Intergration Request - Mapping Templates - application/json
```json
{
    "TableName": "test",
    "Key": {
        "test_id": {
            "S": "$input.params('test_id')"
        }
    },
    "UpdateExpression": "set value = :val1",
    "ExpressionAttributeValues": {
        ":val1": {"S": "$input.path('$.value')"}
    },
    "ReturnValues": "ALL_NEW"
}
```

## DELETE
* Intergration type: `AWS Service`
* AWS Region: `ap-northeast-2`
* AWS Service: `DynamoDB`
* HTTP method: `POST`
* Action: `DeleteItem`
* Execution role: 3번에서 만든 `Role의 ARN`
* Intergration Request - Mapping Templates - application/json
```json
{ 
    "TableName": "test",
    "Key": {
      "test_id": { 
        "S": "$input.params('test_id')"
      }
    }
}
```

**주의점** <br/>
`PutItem`과 `UpdateItem`은 크게 차이가 있다. `PutItem`은 해당 `PK`를 잡고 데이터를 `replace`해주고 `UpdateItem`은 정의한 컬럼값만 `set`해준다.
{: .notice--info}


완성하였다. 굳이 AWS Lambda를 쓰지 않아도 DynamoDB CRUD API를 만들 수 있었다.
배포는 Deploy API를 통해서 진행하면 된다.

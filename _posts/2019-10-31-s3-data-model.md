---
title: "AWS: s3의 데이터 모델"
excerpt_separator: <!--more-->
categories:
  - AWS
  - S3
tags: 
  - aws
  - s3 
---

# Amazon S3(Simple Storage Service)
용량에 관계 없이 파일을 적재 가능하고, 99.9%의 내구성을 보장한다.
- 저장 용량이 무한대
- 파일 저장에 최적화
- Version Control 기능 내장

---
## Data Model
### Flat structure
S3 데이터 모델은 flat structure라서 버킷에 계층이나, 폴더가 없다. AWS 콘솔 UI에서는 디렉토리가 존재하는 것 처럼 보이나 실제론 일반적인 파일시스템과 다르다.

<!--more-->

예를 들어, 일반적인 파일시스템에서는 상위 계층을 생성치 않으면, 해당 계층에 파일을 쓸 수가 없다.
```bash
echo 'test' > depth1/test.txt
-bash: depth1/test.txt: No such file or directory
```
하지만, S3에서는 버킷내에 depth1이 존재하지 않아도, 다음과 같이 가능하다.
```bash
awscli s3api put-object --bucket {bucket_name} --key depth1/test.txt --body test.txt
{
    ETag: '1q2w3e4r'
}
```

#### 문제는 없을까?
S3는 파일을 무제한으로 적재할 수 있는 장점이 있는 만큼 side effect가 존재하는데, 
바로 S3 list-objects API에는 최대 1000개의 max-item 제한이 있기 때문이다.

예를 들어, 우리가 데이터를 루트 경로(`/`)에 약 50만개의 파일을 적재 했다고 했을 때, API의 리턴값은 다음과 같다.
```bash
{
    'Contents': [
        {
            "Key": "test1.txt"
            ...
        },
        {
            "Key": "test2.txt"
            ...
        },

        ...

        {
            "Key": "test1000.txt"
            ...
        },
    ],
    "NextToken": "1q2w3e4r"
}
```
1000개의 오브젝트만 조회가 가능하고, `NextToken`기반으로 재조회를 해야한다. 그리고 이 작업을 500번 반복해야한다.

#### 좋은 네이밍 전략은 곧 좋은 퍼포먼스로 이어진다.
따라서 파일의 키를 세분화하여 적재를 하는 것이 바람직하며, 하나의 키에 1000개를 초과하는 키가 적재되지 않도록 아키텍처링을 진행할 필요가 있다.
```bash
/2019/12/13/16h/test.txt
/category/cafegory_number/product/big/main/test.img
```
다음과 같이 적재하는 것 만으로도, `--prefix`기능을 이용해 특정 키로 시작하는 오브젝트를 선별해낼 수 있다.

구현할 서비스의 아키텍처에 맞는 S3 네이밍을 결정하자.

---
title: "AWS: CloudFront 설정 훑어보기"
excerpt_separator: <!--more-->
categories:
  - AWS
  - CloudFront
tags: 
  - aws
  - cloudfront
  - cf
---

# Amazon CloudFront
글로벌 각 리전에 캐시서버가 존재해, 빠르고 고도로 안전하며 프로그래밍 가능한 CDN(콘텐츠 전송 네트워크) 서비스

---

## Create Distribution
### Origin Settings
Origin Domain Name
: CloudFront가 실제로 바라보게 될, Origin(서버 혹은 서비스) 값, e.g. `{bucket}.s3.amazonaws.com`

Origin Path
: 해당 path를 추가해주면, CloudFront가 바라보는 Origin의 해당 path가 기준이 된다(해당 경로를 루트로 보고 적용된다), e.g. `{bucket}.s3.amazonaws.com/{origin_path}`

Origin ID
: 자동으로 생성되며, 해당 CF를 구분짓는 고유 ID이다.

Origin Custom Headers
: 말그대로 커스텀한 헤더가 추가가능하다.
<!--more-->

---
## Default Cache Behavior Settings
### Path Pattern
Viewer Protocol Policy
: 일반적으로 HTTP와 HTTPS를 모두 허용해준다.
- HTTP and HTTPS
- Redirect HTTP to HTTPS
- HTTPS Only

Allowed HTTP Methods
: 허용할 HTTP Method를 선택할 수 있다.
- GET, HEAD
- GET, HEAD, OPTIONS
- GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE

Field-level Encryption Config
: 필드 레벨의 암호화 구성이 가능하다.(아직 사용해보지 못했다)

Cached HTTP Methods
: 캐시 되는 HTTP 메소드

Cache Based on Selected Request Headers
: 프라이빗한 캐시를 구성할 때 사용한다, e.g. `header의 user_id값을 기준으로 캐싱을 분리`

Object Caching
: origin의 객체 혹은 파일에서, 캐시를 제어할 경우 (cache-control: max-age) 사용하며, CloudFront에서 중앙관리를 위해선 일반적으로 Customize를 사용한다.
- Use Origin Cache Headers
- Customize

Minimum TTL
: 최소 TTL

Maximum TTL
: 최대 TTL

Default TTL
: 기본 TTL

**NOTE** <br/> [해당 TTL의 적용기준은 origin의 헤더에 cache-control의 유무에 따라 적용 기준이 다르다. (참고링크)](https://docs.aws.amazon.com/ko_kr/AmazonCloudFront/latest/DeveloperGuide/Expiration.html)
{: .notice--info}

Forward Cookies
: 요청 URL에 사용자 쿠키를 포함할 것인지를 선택한다. (Amazon S3는 해당사항 없음)

Query String Forwarding and Caching
: 쿼리스트링을 캐시 구분자로 이용할 것인지를 뜻한다, e.g. `{cloudfront_endpoint}/index.html?version=1.0`

Smooth Streaming
: 온디멘드 비디오 스트리밍에 사용된다.

Restrict Viewer Access (Use Signed URLs or Signed Cookies)
: IAM의 CloudFront 키페어를 사용하여 서명된 URL에만 콘텐츠 엑세스 권한을 열어 줄것인지를 뜻한다.

Compress Objects Automatically
: 객체 압축을 자동으로 진행할 것인지를 뜻하는데, 해당 기능이 적용 되려면, 요청 헤더에 `Accept-Encoding: gzip`가 추가되어야한다.

Lambda Function Associations
: Lambda@Edge를 여기에 물려줄 수 있다.

---

## Distribution Settings
Price Class
: 어떤 가격 정책을 이용할 것인지다. 선택지에 따라 캐싱되는 리전이 상이하다.
- Use Only U.S., Canada and Europe
- Use U.S., Canada, Europe, Asia, Middle East and Africa
- Use All Edge Locations (Best Performance)

Supported HTTP Versions
: 말대로 서포트 될 HTTP 버전이며, 일반적으로 HTTP2가 빠르다.
- HTTP/2, HTTP/1.1, HTTP/1.0
- HTTP/1.1, HTTP/1.0

Default Root Object
: S3의 정적 호스팅 개념처럼, `origin_path`를 기준으로 어떤 오브젝트를 Default로 보여 줄 것인지 뜻한다.

Logging
: 로그를 사용여부.

Bucket for Logs
: 로그를 적재할 버킷을 선택할 수 있다.

Log Prefix
: 로그의 접두사를 정할 수 있다.

Cookie Logging
: 쿠키도 로킹에 포함 할것인지.

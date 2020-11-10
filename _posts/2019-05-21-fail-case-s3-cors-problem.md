---
layout: post
title:  "금주의 AWS 실패사례 - S3 Bucket에 업로드를 할 때 CORS 에러 발생"
date:   2019-09-26 03:00:00 +0900
categories: AWS S3
comments: true
---

# 이번 주 AWS를 사용하면서 겪은 실패 사례
* S3 Bucket에 업로드를 할 때 자꾸 CORS 에러 발생
AWS console이나 CLI 등이 아닌 Web browser에서 HTTP request로 S3에 파일을 업로드 하려고 할 때 CORS(Cross-Origin Resource Sharing) 위반 에러가 뜬다.  

## 실패 사례
* JavaScript를 통해 S3로 바로 업로드를 하는 기능을 개발하고자 함
* AWS Signiture Version 4 이용, S3에 업로드하기 위한 각종 token과 정보를 받아서 처리
* Bucket 주소에 POST request를 보냄
* CORS 에러 발생
  + Browser의 ErrorMessage
  ~~~
    Access to XMLHttpRequest at 'https://s3.amazonaws.com/your-resources-bucket/' from origin 'https://www.yoursite.com' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.

    Cross-Origin Read Blocking (CORB) blocked cross-origin response https://s3.amazonaws.com/your-resources-bucket/ with MIME type application/xml. See https://www.chromestatus.com/feature/5629709824032768 for more details.
  ~~~

  + 보내는 Request(POST) Headers
  ~~~
    Accept: */*
    Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryvAQGrBlzxxxxxxxx
    Origin: https://www.yoursite.com
    Referer: https://www.yoursite.com/item/799c5f4a-08eb-4c86-be45-3c782af00000
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.157 Safari/537.36
  ~~~

  + 응답이 안옴 (301에러 301 Moved Permanently)
    - POST가 아니라 그냥 GET https://s3.amazonaws.com/your-resources-bucket/item.jpg 처럼 요청해도 301에러가 반환됨

  + 만약 성공하면 이런 Response Headers가 와야함
  ~~~
    Access-Control-Allow-Methods: GET, POST, PUT
    Access-Control-Allow-Origin: *
    Access-Control-Expose-Headers: Access-Control-Allow-Origin
    Access-Control-Max-Age: 3000
    Date: Mon, 20 May 2019 13:52:11 GMT
    ETag: "a3c98b5e40324972487f757a00000000"
    Location: https://s3.amazonaws.com/your-resources-bucket/item.jpg
    Server: AmazonS3
    Vary: Origin, Access-Control-Request-Headers, Access-Control-Request-Method
    x-amz-id-2: 5rCEIlAaJ8s31IQ97xLPAnArnQgJUm0O0NAHpzDo8IxxxxxOaxFbL9yxSv3ON9kdoxxxxxxxxxxx=
    x-amz-request-id: F7BE545429FFFFFF
  ~~~
* S3 Bucket의 CORS 설정
  ~~~ xml
  <?xml version="1.0" encoding="UTF-8"?>
  <CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
  <CORSRule>
      <AllowedOrigin>*</AllowedOrigin>
      <AllowedMethod>GET</AllowedMethod>
      <AllowedMethod>POST</AllowedMethod>
      <AllowedMethod>PUT</AllowedMethod>
      <MaxAgeSeconds>3000</MaxAgeSeconds>
      <ExposeHeader>Access-Control-Allow-Origin</ExposeHeader>
      <AllowedHeader>Authorization</AllowedHeader>
  </CORSRule>
  </CORSConfiguration>
  ~~~

## 문제 해결 과정
* I think your CORS setting should work. My suspicion is that you preflight request was blocked somewhere. Are you accessing the bucket that in the same region as your client?
  - Pre-flight가 안되는거 아닌가?
  - https://github.com/aws/aws-sdk-js/issues/1898

* Region에 대해 우려를 함
  - https://stackoverflow.com/questions/41111244/cors-error-with-s3-even-after-adding-cors-configuration

* Pre-flight 를 확인 해 보길 원함
  - Access-Control- 로 시작하는 header가 response에 있는지?
  - https://stackoverflow.com/questions/32839310/amazon-s3-direct-upload-cors-error

## AWS의 Region별 endpoint가 원인일까?
* S3는 endpoint 체계가 좀 복잡함
  - S3가 발전하면서 S3의 주소가 많이 바뀌었는데, 하위 호환을 위해 옛 버전의 endpoint도 유지되고 있음
  - S3는 global 서비스이지만 bucket은 region별로 존재하는 다소 헷갈리는 상황
  - 이로 인해 region에 따라 endpoint가 다름, 근데 region과 상관없는 통합(?) endpoint도 존재했었음(오래된 일이라 확실하지 않음)
  - 즉, 옛날 코드에서 S3 endpoint를 "s3.amazonaws.com"로 쓰고 있었음
* Endpoint를 최신으로 변경하니 해결
  - 참고: [AWS service endpoints](https://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region), [AWS S3 endpoints](https://docs.aws.amazon.com/general/latest/gr/s3.html)

## 해결 코드
Java로 된 code를 보면 endpoint를 AWS SDK를 통해 안전하게 획득할 수 있음
  ~~~ java
    // Enable CORS
    String bucketName = "your-resources-bucket";
    Region currentRegion = Regions.getCurrentRegion(); // region of the EC2 instance
    AmazonS3 client = AmazonS3ClientBuilder.standard()
    .setRegion(currentRegion.getName());
    .withClientConfiguration(new ClientConfiguration())
    .build()
    BucketCrossOriginConfiguration corsConfig = client.getBucketCrossOriginConfiguration(bucketName);
    List<CORSRule> rules = corsConfig == null ? new ArrayList<>() : corsConfig.getRules();
    rules.add(new CORSRule()
    .withAllowedMethods(ALLOWED_METHODS) // List<CORSRule.AllowedMethods> "GET, POST, PUT..."
    .withAllowedOrigins(Collections.singletonList(serverBaseUrl)) // String
    .withMaxAgeSeconds(MAX_AGE_SECONDS) // int
    .withExposedHeaders(Collections.singletonList("Access-Control-Allow-Origin"))
    .withAllowedHeaders(Collections.singletonList("Authorization")));
    client.setBucketCrossOriginConfiguration(bucketName, new BucketCrossOriginConfiguration().withRules(rules));

    String endpoint = currentRegion.getServiceEndpoint("s3");
    String uploadUrl = String.format("https://%s/%s/", endpoint, bucketName);
    // Ex) https://s3.eu-central-1.amazonaws.com/your-resources-bucket/
  ~~~

## S3 CORS설정 가이드
여기에 모든 답이 있음
- https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/dev/cors.html
- https://docs.aws.amazon.com/AmazonS3/latest/dev/cors.html
- https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/dev/cors-troubleshooting.html

## 기타 참고 자료
* Access-Control-* 관련 header가 오지 않을 때?
  - CORS설정이 되어 있음에도 안온다면 Request에 "Origin" header가 있는지 확인, 있어야 함
  - https://stackoverflow.com/a/48774074/8350542
  
* CORS관련 설명
  - https://homoefficio.github.io/2015/07/21/Cross-Origin-Resource-Sharing/
  - https://dev.to/miguelmota/understanding-cross-origin-resource-sharing-cors-2i3e

* Sprin에서 CORS를 enable하는 방법 - Global하게 적용하기 위해선 WebMvcConfigurer 설정 필요
  - https://spring.io/guides/gs/rest-service-cors/#_enabling_cors
  - https://www.baeldung.com/spring-cors

* CorsRegistry말고 Filter로 처리하는 방법
  - https://stackoverflow.com/questions/40286549/spring-boot-security-cors

* cors-preflight-인증-처리-관련-삽질 글 
  - CORS 처리를 위한 Filter는 반드시 인증 처리하는 Filter 이전에 있어야 한다.
  - https://www.popit.kr/cors-preflight-%EC%9D%B8%EC%A6%9D-%EC%B2%98%EB%A6%AC-%EA%B4%80%EB%A0%A8-%EC%82%BD%EC%A7%88/

* Origin이 없고 Sec-Fetch-Mode만 있을 경우가 있다.
  - Sec-Fetch-Mode 헤더는 무엇인가? 서버가 요청이 쉽게 합법적인지 판단하거나 거부하기 위한 추가 header, preflight 아니고 상호보완을 위한 헤더일 뿐
  - https://stackoverflow.com/a/57679554/8350542
  - Reference: https://www.w3.org/TR/fetch-metadata/#sec-fetch-site-header

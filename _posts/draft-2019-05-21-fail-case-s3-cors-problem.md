---
layout: post
title:  "금주의 AWS 실패사례 - S3버킷에 upload를 할 때 자꾸 CORS 에러 발생"
date:   2019-09-26 03:00:00 +0900
categories: AWS S3
comments: true
---

# S3 Bucket에 upload를 할 때 자꾸 CORS 에러 발생

## S3 CORS설정 가이드
https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/dev/cors.html
https://docs.aws.amazon.com/AmazonS3/latest/dev/cors.html
https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/dev/cors-troubleshooting.html


## 실패 사례
* Browser의 ErrorMessage
  ~~~
    Access to XMLHttpRequest at 'https://s3.amazonaws.com/your-resources-bucket/' from origin 'https://www.yoursite.com' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.

    Cross-Origin Read Blocking (CORB) blocked cross-origin response https://s3.amazonaws.com/your-resources-bucket/ with MIME type application/xml. See https://www.chromestatus.com/feature/5629709824032768 for more details.
  ~~~

* 보내는 Request Headers
  ~~~
    Accept: */*
    Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryvAQGrBlz3hquFLpZ
    Origin: https://www.yoursite.com
    Referer: https://www.yoursite.com/item/799c5f4a-08eb-4c86-be45-3c782af32809
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.157 Safari/537.36
  ~~~
* 응답이 안옴 (301에러 301 Moved Permanently)
  - POST가 아니라 그냥 GET https://s3.amazonaws.com/your-resources-bucket/item-799c5f4a-08eb-4c86-be45-3c782af32809.jpg 이렇게 붙어도 301에러가 반환됨

* 만약 성공하면 이런 Response Header가 옴
  ~~~
    Access-Control-Allow-Methods: GET, POST, PUT
    Access-Control-Allow-Origin: *
    Access-Control-Expose-Headers: Access-Control-Allow-Origin
    Access-Control-Max-Age: 3000
    Date: Mon, 20 May 2019 13:52:11 GMT
    ETag: "a3c98b5e40324972487f757a000000af"
    Location: https://s3.amazonaws.com/your-resources-bucket/item-799c5f4a-08eb-4c86-be45-3c782af32809.jpg
    Server: AmazonS3
    Vary: Origin, Access-Control-Request-Headers, Access-Control-Request-Method
    x-amz-id-2: 5rCEIlAaJ8s31IQ97xLPAnArnQgJUm0O0NAHpzDo8IxxxxxOaxFbL9yxSv3ON9kdoGCvL7lyTm4=
    x-amz-request-id: F7BE545429FFFFFF
  ~~~

## 문제 해결 과정
* I think your CORS setting should work. My suspicion is that you preflight request was blocked somewhere. Are you accessing the bucket that in the same region as your client?
  - Pre-flight가 안되는거 아닌가?
  - https://github.com/aws/aws-sdk-js/issues/1898

* Region에 대해 우려를 많이들 함
  - https://stackoverflow.com/questions/41111244/cors-error-with-s3-even-after-adding-cors-configuration

* Pre-flight 를 확인 해 보길 원함
  - Access-Control- 로 시작하는 header가 response에 있는지?
  - https://stackoverflow.com/questions/32839310/amazon-s3-direct-upload-cors-error

* Access-Control-* 관련 header가 오지 않을 때?
  - CORS설정이 되어 있음에도 안온다면 Request에 "Origin" header가 있는지 확인, 있어야 함
  - https://stackoverflow.com/a/48774074/8350542

## AWS의 Region별 endpoint가 원인일까?
* S3는 endpoint 체계가 좀 복잡함
  - S3가 발전하면서 S3의 주소가 많이 바뀌었는데, 하위 호환을 위해 옛 버전의 endpoint도 유지되고 있음
  - S3는 global 서비스이지만 bucket은 region별로 존재하는 다소 헷갈리는 상황
  - 이로 인해 region에 따라 endpoint가 다름, 근데 region과 상관없는 통합(?) endpoint도 존재
* 참고: [AWS service endpoints](https://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region), [AWS S3 endpoints](https://docs.aws.amazon.com/general/latest/gr/s3.html)

## 이 때 어떻게 해결했는지 기억이 안나....

## 해결
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

        String uploadUrl = String.format("https://s3.%s.amazonaws.com/%s/", currentRegion.getName(), bucketName);
        // Ex) https://s3.eu-central-1.amazonaws.com/your-resources-bucket/
    ~~~

## 기타 참고 자료
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

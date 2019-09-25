---
layout: post
title:  "금주의 AWS 실패사례 - S3버킷에 upload를 할 때 자꾸 CORS 에러 발생"
date:   2019-09-26 03:00:00 +0900
categories: AWS S3
comments: true
---
# ElasticBeanstalk에 SpringBoot 배포를 하자

S3버킷에 upload를 할 때 자꾸 CORS 에러 발생

S3 CORS설정 가이드
https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/dev/cors.html
https://docs.aws.amazon.com/AmazonS3/latest/dev/cors.html
https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/dev/cors-troubleshooting.html

I think your CORS setting should work. My suspicion is that you preflight request was blocked somewhere. Are you accessing the bucket that in the same region as your client?
https://github.com/aws/aws-sdk-js/issues/1898

리전에 대해 우려를 많이들 함
https://stackoverflow.com/questions/41111244/cors-error-with-s3-even-after-adding-cors-configuration

Pre-Flight 를 확인 해 보길 원함
https://stackoverflow.com/questions/32839310/amazon-s3-direct-upload-cors-error


https://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region


Browser ErrorMessage

Access to XMLHttpRequest at 'https://s3.amazonaws.com/prod-sqe-appcenter-resources/' from origin 'https://appcenter-test.cloudprintsolutions.com' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.

Cross-Origin Read Blocking (CORB) blocked cross-origin response https://s3.amazonaws.com/prod-sqe-appcenter-resources/ with MIME type application/xml. See https://www.chromestatus.com/feature/5629709824032768 for more details.


- 보내는 Request Headers
Accept: */*
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryvAQGrBlz3hquFLpZ
Origin: https://appcenter-test.cloudprintsolutions.com
Referer: https://appcenter-test.cloudprintsolutions.com/management/apps/requested/view/799c5f4a-08eb-4c86-be45-3c782af32809
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.157 Safari/537.36

- 성공하면 이런 Response Header가 옴
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Origin: *
Access-Control-Expose-Headers: Access-Control-Allow-Origin
Access-Control-Max-Age: 3000
Date: Mon, 20 May 2019 13:52:11 GMT
ETag: "a3c98b5e40324972487f757a07c122af"
Location: https://s3.amazonaws.com/dev-dp-appcenter-resources/forum%2Fd6a92288-2317-45f4-a4f0-a77a190e026a%2F46087.jpg
Server: AmazonS3
Vary: Origin, Access-Control-Request-Headers, Access-Control-Request-Method
x-amz-id-2: 5rCEIlAaJ8s31IQ97xLPAnArnQgJUm0O0NAHpzDo8I2MvayOaxFbL9yxSv3ON9kdoGCvL7lyTm4=
x-amz-request-id: F7BE5454299B9EAF

- 실패하면 안옴 (301에러 301 Moved Permanently)

그냥 GET https://s3.amazonaws.com/prod-sqe-appcenter-resources2/ 이렇게 붙어도
301에러가 반환됨

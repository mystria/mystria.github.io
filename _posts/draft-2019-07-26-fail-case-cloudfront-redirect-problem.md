---
layout: post
title:  "금주의 AWS 실패사례 - CloudFront의 문제점"
date:   2019-07-26 20:00:00 +0900
categories: Linux
comments: true
---
# CloudFront의 고질적인 문제
CloudFront에 S3 object를 OAI를 통해 배포하고자 하였는데, 정상 배포했음에도 불구하고 자꾸 S3 주소로 Redirect 되어 Access Denied 문제가 발생하였다.  
CloudFront Origin 설정이 문제일까, S3 Policy가 문제일까 한참 확인하고 재확인 했음에도 설정은 문제 없는데...  
알고보니 CloudFront에 고질적인(?) 문제가 있었다.  

##
CloudFront의 오리진에서 S3 버킷의 기본 주소(??.s3.amazonaws.com)을 리전별 공식(?)주소로 바꿔줘야 하는데, 이 부분이 상당히 오래 걸림
처음부터 기본 리전(오레곤?)에 생성하면 문제 없음
가끔 바로 될 때도 있으나, 어떤 때는 며칠씩 걸리기도 하는 듯...
해결 책으로는 기다리거나....
S3 오리진 선택시 리전별 주소로 직접 입력하면 됨

https://forums.aws.amazon.com/thread.jspa?threadID=216814
https://forums.aws.amazon.com/message.jspa?messageID=677491#677491


https://stackoverflow.com/a/38736282/8350542


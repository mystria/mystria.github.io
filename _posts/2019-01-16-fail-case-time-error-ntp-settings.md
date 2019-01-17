---
layout: post
title:  "금주의 실패사례 - 시간 관련 인증 오류"
date:   2019-01-16 21:00:00 +0900
categories: NTP Boto3 Failure
comments: true
---

# Python Boto3 실행 시 의문의 에러
Python3 + Boto3 를 이용하여 동작하는 AWS 운영 스크립트가 있었다.  
잘 돌아 가던 녀석이었는데, 2개월 만에 다시 돌렸더니 오류가 발생했다. 왜일까?  

## SignatureDoesNotMatch
* 아래와 같은 메시지 발생
> An error occurred (SignatureDoesNotMatch) when calling the DescribeConfigurationSettings operation: Signature not yet current: 20190116T101413Z is still later than 20190116T101330Z (20190116T100830Z + 5 min.)
* SignatureDoesNotMatch는 무엇일까?
  - Signature 는 보통 함수의 parameter를 의미하는것 아니었나?
  - 그런데 왠 시간?

## 인증 문제
* Google 검색결과, credentials과 관련이 있는것 같음
  + https://github.com/aws/aws-cli/issues/602
  + https://github.com/aws/aws-cli/issues/2665
* 직접 AWS CLI를 호출해 봄
  + 역시 AuthFailure 발생
  > An error occurred (AuthFailure) when calling the DescribeInstances operation: AWS was not able to validate the provided access credentials
  + 뭐가 문제지?

## 해결
* 실행하려는 환경의 시간과 AWS 시간의 차이가 5분이상 나서 그런것이었다!
* 로컬 환경의 시간을 수동으로 설정했었는데, 어찌된 일인지 느려짐
* NTP(Network Time Protocol) 설정으로 해결하자
  + Network 시간과 동기화하면 안심
  + 본인은 Ubuntu 사용자이므로 아래 링크로 해결함
    - http://webdir.tistory.com/208
    - https://www.digitalocean.com/community/tutorials/how-to-set-up-time-synchronization-on-ubuntu-16-04
  ~~~ sh
  $ timedatectl  // 현재 시간 설정 확인
  $ ntpq -p      // NTP 서버 연결상태 확인
  ~~~


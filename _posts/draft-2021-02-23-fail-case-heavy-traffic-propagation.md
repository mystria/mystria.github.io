---
layout: post
title:  "금주의 실패사례 - Heavy traffic 때문에 발생한 timeout 해결"
date:   2021-02-23 23:00:00 +0900
categories: EC2
comments: true
---
# Heavy traffic 때문에 발생한 timeout 해결

## 구조
* Front-end - EC2(Backend) - ALB-EC2(Hosted Apigee) - ALB-EC2(Service) - Managed MongoDB
*             다른 서비스 -----

## 원인
* 다른 서비스의 초당 수백번 API 호출

## 분석 및 해결
* 1차 문제 발생
  1. MongoDB의 IOPS가 급증
    - 많은 request로 DB 생성 및 조회 급증 -> IOPS 증가
    - MongoDB의 응답이 느려져 Service EC2 에서 60초 넘게 지연 발생
    - Backend EC2 에서 60초 안에 응답을 못 받아 timeout 에러 발생
    - Service EC2 앞의 ALB의 monitoring에서 response time 확인 시 Maximum 60s를 넘는 traffic이 확인 됨
  2. 해결
    - MongoDB의 성능 업그레이트 (tier를 높여 최대 IOPS를 확장)

* 2차 문제 발생
  1. Backend EC2에서 60초 안에 response 데이터를 다 못받음
    - MongoDB가 개선되어 Service와 Apigee에서의 response time 은 정상
    - 즉, Service 에서는 이상 없음
    - 그런데 Backend에서 data를 다 못받음
      - log에 read time out 메시지와 array 관련 정보가 표시 됨
    - 네트워크 대역폭(또는 EC2의 network out/network packets out)의 문제로 판단
    - Service에서 응답이 빨라지자 그만큼 request 처리량이 늘어났기 때문
  2. 해결
    - Apigee EC2를 늘여 traffic 분산

## 생각할 점
* 관리형 서비스인 ELB(ALB)에서는 문제가 없음
* Apigee말고 Amazon API Gateway였으면 어땠을까?
* DB를 관리형으로 쓰니 원인 확인 및 해결되 쉬움
* 관리형이 답?!

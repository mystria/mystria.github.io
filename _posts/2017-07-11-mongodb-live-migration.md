---
layout: post
title:  "MongoDB Atlas Live Migration 방법"
date:   2017-07-11 19:00:00 +0900
categories: MongoDB Migration Import
comments: true
---
# MongoDB Atlas의 Live Migration(Import) 방법

## MongoDB Atlas
- MongoDB Atlas는 MongoDB 관리형 서비스(SaaS)
- Local 또는 Cloud에 운영 중인 MongoDB를 Atlas로 최소한의 down time으로 옮길 수 있음
- 유료 Atlas cluster만 Live Migration 지원함
- 본 글은 MongoDB의 [Lightning Demo][mongodb-atlas-lightning-demo]를 참고하여 간단하게 요약한 것

## 준비사항
- source mongodb를 3.0이상으로 업그레이드(Atlas가 3.2/3.4만 지원하고, 3.0 이상이 live migration을 지원)
- source mongodb보다 큰 Atlas storage
- source mongodb의 방화벽이 Atlas를 허용하고, Atlas도 source mongodb를 허용하게 WhiteList설정
  - 당연하지만 Application이 Atlas를 접속할 수 있게 Atlas security 설정 필요
- source mongodb의 username, password, CA file등을 미리 준비

## 진행순서
- Atlas의 Clusteres에서 migration 할 target mongodb cluster를 선택
- 추가기능 버튼(... 버튼)을 누르고 Migrate Data to this cluster 클릭
- 설명서가 나옴(체크리스트 확인)
- I'm ready to migrate
- source mongodb의 host:port와 username, password, CA 입력하고 Validate로 연결 체크
- Start Migration
- (한참 진행이 되면)초록색 타이머가 켜지면서 cut over준비
  - 메일로 준비되었다고 알림이 옴
  - (72시간 동안 라이브 마이그레이션 진행됨, 이후에는 새로 해야됨)
- 타이머가 0:01쯤이면 사실상 source mongodb와 동일한 수준으로 리얼타임으로 동기화 중, cut over가능
- Start Cutover
- 안내창이 뜨고 또 다른 시계가 표시. 이게 0:00되면 완전이 동일한 것
- Cut over할 Application을 멈추면 시계도 곧 0:00이 됨(완료되면 옆에 체크가 표시됨)
  - Application이 멈춰 더 이상 신규 data가 생기지 않으므로 완전히 동일해 지는 것
- 새로운 mongodb의 cluster 주소를 Application에 덮어 쓰고
- I'm Done
  - (이 버튼 누르면 더 이상 source mongodb와 동기화 하지 않음)
- Application 재시작
  - Application 재부팅 하는시간 만큼 잠깐 서버가 끊기기는 함

## 끝
[mongodb-atlas-lightning-demo]: https://www.mongodb.com/presentations/lightning-demo-mongodb-atlas-live-import-fri

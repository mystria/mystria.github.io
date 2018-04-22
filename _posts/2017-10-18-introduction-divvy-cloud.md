---
layout: post
title:  "DivvyCloud 소개"
date:   2017-10-18 18:00:00 +0900
categories: DevOps
comments: true
---
# DivvyCloud 소개
본 포스팅은 2017년 10월 18일 서울 강남에서 있었던, DivvyCloud 소개 세미나 참석 후, 내용을 개인적인 목적으로 정리한 것 입니다.  
Cloud가 대중화 되면서 기업에 cloud 관리용 도구를 제공하는 업체들도 많이 등장하였습니다. Divvy Cloud도 그 중 하나로, Enterprise급 기업에서 public cloud를 관리할 때 발생하는 문제(Startup과는 달리 여러 내외부용 서비스를 운용)에 대한 해결책을 제시하고 있습니다.  
이 글은 홍보 목적이 아닙니다. 업체로부터 아무것도 제공받은 바가 없습니다. 그래서 이 글도 6개월 뒤에 공개 포스팅 합니다.

## 대용량 Amazon 환경 대상
  * GE에서 9000개의 application을 AWS로 옮기는 작업 중에서 일어난 경험을 기반으로 만들어짐
  * 대량의 조직/앱을 옮기고 이를 유지관리하는 방법에 대해 설명
  * Cloud 관리 고려사항
    + Cost
    + Compliance/Security
    + Culture
  * 첫번째로 필요한 Mindset: 자신의 상태를 파악하는 것
    + 여러분의 환경은 AWS의 BestPractice나 Netflix의 케이스랑 다르다.
    + Netflix(와 같은 대부분 스타트업)는 하나의 앱에 수만명의 사용자
    + 그러나 대부분의 기업은 수백개의 앱과 흩어진 사용자들
    + 내부적으로도 쓰고(AD 같은..)
    + 엔터프라이즈의 (법적)규제와 보안을 고려해야 함
    + 미국에서도 여러 보안 사고 발생, S3 노출 사고도 많음
      - 하나의 방식으로는 모두를 맞출 수 없음 -> 그래서 DivvyCloud

## 보안
DivvyCloud란? Divide의 줄임말
인프라를 워크로드나 앱 기준으로 분할하고, 앱/팀/프로젝트 별로 분할한 것을 각각의 보안 룰을 다르게 적용
  * 고객사례를 기반으로 보면
    + 전체적으로 적용되는 룰(Global rule)/ BU(Business Unit)별로 적용되는 기준이 있다.
    + Vertical에 따라 정해진 법/규칙이 다르다.
    + 어떻게 보안을 관리할 지는 *Global Rule/특정(Local) Rule의 합쳐진 형태* 이다.

  * 보안의 첫번째 요소는 Get Global Visibility
    + AWS를 크게 사용하는 고객은 대부분 가진 문제: 전체를 한 눈에 볼 수 없음
    + 관리 중인 계정(AWS Account)이 몇 개인지 모름
      - 부서별 Account를 Best Practice로 제시 -> 이게 원인 중 하나
    + Region 단위로 조회/관리
      - 대부분 AWS Management Console에 로그인하여 관리하기 때문 -> Region단위로 보여줌
    + 3가지 관리 지점
      - AWS Accounts
      - Regions
      - Resources
    + 한번에 볼수 있는 Dashboard같은게 필요 -> Visibility 확보
  * 그 다음엔 Tagging이 필요
    + 보통 5개 : Owner, Application on here, Stage(Dev/Test/Prod...), SecurityLevel, Who Paid
    + 위는 fundamental, 더 쓰는 경우도 있다.
    + Bad Case: Tag가 없는 인스턴스가 엄청 많음
      - Tag가 없는게 제일 큰 포션
      - 구분할 수 없고
      - 보안을 알 수 없고
      - 누구껀지 애매하고
      - 비용이 어디로 나가는지 모름
    + 해결책: 2 step
      - 메일을 전체 사원에게 보냄 -> 제한시간 이후 모두 Termination.
      - 향후 생성되는 인스턴스에 Tag가 없으면 무조건 종료시킴
      - 이렇게 하면 처음엔 AWS문제인줄 알고? 두번째부터 의문을 갖고, 세번째 부터 IT에 문의함
      - 두 달 정도 후에 Tagging 전략을 정착시킴
      - 강제적으로 적용해야하고, 즉시 적용해야 함!
      - 대부분의 개발자들이 반응을 하지 않으므로 보다 공격적으로 적용해야 함

  * 자동화
    + AWS의 보안 요소가 많다.
    + AWS의 Feature가 계속 늘어난다. -> 보안관련 이슈 증가
    + 기존 보안과 더불어 새로운 보안이 늘어나므로 복잡성이 증가
    + 인프라 시스템 자체의 문제 뿐 아니라 앱, 또는 사용자에 의한 보안 문제가 생길 수 있따.

    + 자동화가 중요한 이유는, 위 요소들을 매일 관리하기에 부담이 된다.
    + 고객과 이를 해결하기 위해 고려한 것은?
      - (위에 언급했던) Global Policy, Local Policy의 조합

## 문화적인 요소
  * 가장 어려운 파트
  * Cloud를 사용하는 것은 싸지는 것 같지만, 사실은 싸질 가능성이 있다는 뜻
    + 즉 올바르게 사용해야 함
  * Cloud에 대한 마인드 셋 전환이 필요
    + scale up vs scale out
    + scale out 의 경우 사이즈 조절 가능 - 비용절감, HA, 등…
  * Cloud의 영향
    + One-way journey(일방향 여행): Amazon으로 가면 돌아오지 못함, 왜냐? 민첩성
    + 사용자(개발자)들에게 클라우드 사용 권한을 주는 것이 제일 효율적(most value)
      - 그러나 risky로 대부분 거부감, 보통 중간에 포털을 두고 요청에 따라 생성해 줌
      - 이것이 잘 동작하는 것을 본 적이 없음 -> 사용자에게 접근권한을 충분히 주고 사후 관리 하는게 더 낫다는 결론을 내림
    + 조직별로 적합한 툴이 필요: 툴이 조직에 맞춰지도록..

## DivvyCloud가 배운 것들
규칙을 강제하지 말고, 툴을 조직에 맞추고, 사용자들이 일을 하도록 도와주고, 자동화를 통해 확장성/복잡성을 통제하는 체제를 갖춰야 함.  
  * 어떻게?!
  * DivvyCloud의 제안
    + 순환 프로세스(항상 돌고 있음)
    + Harvest -> Unify -> Analysis -> Action
      - Harvest : 어카운트, 리전, 서비스 별로 정보 수집
      - Unify : 위 정보를 하나에 모아두고(Global Inventory)
      - Analysis : 분석하여 규칙을 무너뜨리는 것을 찾고
      - Action : 해결
  * 3가지 카테고리
    + 비용 Cost : AWS에서는 3가지 간단한 방법
      - Prod아니면 resource를 shutdown(Schedule 또는 여러방법으로)
      - Orphaned resource 정리 - 미사용 리소스 정리(EC2없는 EBS나, 접속이 없는 EC2 등)
      - Data Retention - 오래된 데이터 정리...
    + 보안 Security 3가지 집중포인트
      - 네트워크 레벨 보안, Security Rule에 대한 audit: 임시로 열었던 포트를 안닫는 경우 많음
      - 사용자 계정: 개발하다 권한 막힌다고 다 열어줬는데, 이 계정이 잘못된 패스워드 정책일 경우… 폭망
      - 데이터 보안: Bad Permission of S3(Open public when create with AWS Console wizard) 등..
    + 표준화, 규제 Rule and Standard(Policy compliance)
      - Tag Validation
      - Regional Audit: 지역 법 준수
      - Instance Audit: AMI가 어떤건지 알 수 없다. 개발자들은 비용생각 안하고 엄청 큰걸로 마구 생성

## 정리: DivvyCloud가 하는 일
Agent 기반(뭔가를 설치해야 하는) 관리 정책은 별로 좋지 않다고 생각, 그래서 Divvy는 Cloud API기반 관리
  * Account를 등록하면 해당 계정의 모든 리소스를 확인
  * 필터를 통해 문제(?)가 있는 리소스를 리스팅해주고, 필터의 조합을 인사이트로 하여 저장해서 dashboard화
  * 그리고 여기에 해당하는 것들을 자동으로 찾는 Bot으로 만들 수 있음(대상지정, 조건지정, 처리방법 지정, 동작시간 지정)
  * 처리방법에는 notify도하고 백업 후 삭제도 하고 여러가지가 가능함
  * 이런 봇이 언제 동작할지도 지정가능(Reactive(특정 상황 발생시), On-Demand(요청하면), Scheduling(지정된 시각에))
  * 또한 다양한 인사이트를 DivvyCloud store(?)에서 받아 적용할 수 있음
  * 비용은 규모별로 나눠져 있지만 custom으로 kr@divvycloud.com

---
layout: post
title:  "AWS Solutions Architect 교육과정 3일차"
date:   2019-06-24 22:00:00 +0900
categories: AWS
comments: true
---
# AWS Solutions Architect 교육과정 3일차 정리
본 글을 2015년 한국 AWS에서 교육받은 내용을 메모해 뒀던 글을 다시 정리하여 올린 글입니다.  
2015년 교육 자료이므로 현 시점(2019년)과는 차이가 있을 수 있습니다.  
용어, 한글표기 등은 시간 관계상 맞추지 않았습니다.  

## Cloud 관리
아마존을 수동 구성하면 힘들다. 재현 할 가능성이 높음  
DIY 스크립트? 커스터마이징이 좋지만 여러 문제가 있음(유지보수, 종속성 등등)
* Cloud Formation
  + 탬플릿(json, 리소스를 코드로 취급하여 저장) -> CloudFormation -> 스택(클라우드 포메이션에서 생성한 리소스 모음)
  + 단점: 배우기 어렵다.
  + json으로 작성해야 하므로 여러 편집기가 지원됨 (VisualOps, CloudFormation Designer)
  + 일반적으로 쓰이는 템플릿이 공유되고 있음
  + 탬플릿을 실행시키면 스택이 만들어짐(가능한 병렬로 수행됨)
  + 탬플릿은 리전이나 계정에 종속되면 안되므로, 고유값(키, 이름)등은 parameter로 정의해두면 탬플릿 실행시 입력을 받음
    - AMI의 경우 리전에 종속되므로 다른 리전에서는 같은 종류의 AMI일지라도 ID가 다름 -> Mapping값으로 경우에따라 사용되는 값을 다르게 할 수 있음
    - 경우에 따라 파라메터를 살짝 바꿔주도록 세부조건을 지정해 줄 수 있음
    - DependsOn을 써서 순서대로 만들도록 지정할 수 있음
    - WaitCondition으로 대기시키고 조건이 충족되면 진행되게 설정 가능
    - Lamda를 통해 AWS외 리소스도 체크해가면서 진행 가능
  + 기존 배치된 서비스를 CloudFormation으로 전환도 가능(Cloud Former), 그러나 살짝 모자르기 때문에 정리필요
* 애프리케이션 관리 서비스
  + ElasticBeanstalk(원클릭 런칭 서비스), OpsWorks(엄청 많은 양의 infra관리, chef사용가능)
  + CloudFormation, EC2는 diy급

## 관리형 서비스
* SQS
  + 병목현상을 막고 소결합을 위해 큐를 사용
  + 성능은 카프카가 갑이지만 AWS를 쓴다면 SQS권장
  + 분산시스템이라서 FIFO를 완전하게 보장하지 못함(최대한 지켜주려고 한다... 만약 순서를 꼭 지키겠다면 어플리케이션단에서 처리 필요)
  + 최소한 1회 = 무조건 1번은간다, 2번갈수도! 어플리케이션단에서 처리 필요
  + 위 두문제를 해결하기위해서는 걍 카프카 쓰세요 ㅠ 개선중
  + 메시지가 바로 안읽어와 질 수 있으므로 롱 폴링 권장
  + 큐에서 dequeue를 할때 자동으로 삭제하지 않음 = 명시적으로 삭제해야함
    - 큐의 데이터를 누군가 가져가면 인비저블 처리, 인비저블일때는 다른 누군가가 가져가지 못함
    - 오랜시간동안 삭제되지 않고 인비저블이 유지되면 비저블로 복구됨 -> 복구된 것을 누가 또 가져가면 첫 인스턴스가 삭제 불가(권한 없음), 즉 실패처리해야함
    - 이렇게 구성한 이유는 EC2가 오토스케일링 되면서 사라지면 유실 될 수 있음을 막기 위함
  + 큐의 갯수를 통해 오토 스케일링을 할 수 있음, 큐가 비었을때는 인스턴스를 0개로 할 수 도 있으니깐, 오토스케일링 그룹의 최소 인스턴스가 0을 허용하는 것임
  + 데드레터큐는 따로 보관하고, 사용자가 처리해주면 됨
  + 기존의 SQS->EC2같은 작업을 Lambda가 처리해줄 수 있음(단 짧고 간단한거: 썸네일 생성등)

* SNS
  + 알림을 설정 : 이메일, HTTP/s, SMS, AmazonSQS, 푸시
  + 최대 256Kb까지 지원하지만(64kb로 4번 보내므로 요금 많이 나옴)
  + SMS나 푸시의 경우 요금이 비싸다
  + S3의 경우 Event Notification을 SNS로 보낼 수 있음, SNS 에서 SQS로 http를 보내고 SQS에서 작업처리 가능
    - 이렇게 복잡하게도 할 수 있지만 S3를 바로 lambda로 처리해버리는 것도 가능
  + SNS가 다른데로 보낼때는 받는 사람이 승인해야 함
    - SQS의 경우 ARN을 등록해야 SNS를 보낼 수 있음
    - e-mail은 받기전에 동의 해야함
    - 비즈니스 용으로 쓸 수도 있지만 보통은 운영중 이슈 확인용으로 쓰는게 좋지 않을까?

* SWF
  + 복잡한 업무를 처리해주는 작업 처리자
  + 정확히 한번 처리함
  + 롤 폴링 - API를 통해 작업상태 추적 가능

* SES
  + 대용량 이메일 전송 서비스
  + 처음엔 발송이 제한이지만 스팸이 아니라고 판단되면 점차 발송 횟수 증가가능
  + 엔터프라이즈는 걍 대량전송 가능

* CloudSearch (검색 엔진 같은거)
  + 간편하게 검색기능을 쓸때 사용 가능
  + 검색은 인덱스를 기반으로 하고 결과는 원본을 가르킴 : 검색 트래픽이 커지면 인스턴스를 복제해서 검색 하게끔 하고, 데이터가 커지면 파티션을 늘여서 검색가능하게 함


## 빅데이터, 대용량 데이터 처리
* Amazon EMR
  + 관리형 Hadoop
  + 참고 (Redshift는 OLAP을 이용한 BI분석)
  + EMR과 Redshift조합을 많이 씀
  + 반구조, 구조화 되지 않은 데이터를 처리할때 사용 : 데이터의 차원을 좀 줄일때 사용

* Redshift
  + 페타바이트단위 관리형 데이터 웨어하우스 서비스
  + 관계형 DB랑 달리 엄청 빠른 액세스가 가능하지만 분석용(DB로는 쓰지 말라네)

* Kinesis
  + 스트림 데이터 처리, 시간당 수백테라바이트 처리
  + 스트림 데이터를 받아서 샤드 단위로 파티션키를 넣어 정리
  + 저장 단위는 샤드, 초당 10Mb처리 : 처리량을 늘이려면 샤드를 증가하면 됨
  + 앱이라 부르는 곳에서 처리(MapReduce작업)
  + 리포트를 보여줄 수 있음(리전기반이지만 Global처럼 사용됨)

## 기타
  * Cognito : 모바일 환경에서 적절한 솔루션 : 사용자 인증, 데이터 싱크
  * Mobile Analytics
  * Elastic Transcoder : S3에 올리면 자동으로 인코딩해주는 서비스 : 건바이건이라서 비쌈
  * Dynamic DynamoDB : Open Source Third party : dynamoDB 를 오토스케일링 해줌(증가 무제한, 감소 일 4회)

## Disaster Recovery
* 내결함성(fault tollerent)
  + 높음 : 멀티 AZ
  + 아주높음 : 교차 리전(스냅샷, AMI 복사 - 데이터 계층을 비동기식 복제(동기식 복제는 거의 불가))
* 고가용성(High availability) : 서버 중지가 발생해야 한다면?
  + S3를 통해 정적 웹사이트 사용
  + DNS장애 조치를 통해 Route53으로 장애안내 페이지로 연결
* RTO / RPO
  + RTO : Recovery time objective : 장애시 얼마나 빨리 복구되나?
  + RPO : Recovery point objective : 장애시 최근 데이터가 손실일어나는 구간을 줄이자
* DDoS 공격을 막는 기술은 전반적으로 적용되어 있음
  + ElasticBeanstalk에도 적용됨
  + 가장 best는 CloudFront (동적인 데이터에도 좋음)
* 장애를 고려한 설계
  + 단일 장애 지점을 제거(복수 구축)
  + 모든 기능은 장애가 발생한다는 것을 가정하여 구축
  + 복구 프로세스! (백업은 다들 하지만 복구는 잘 안하더라) : 복구 프로세스가 정상 동작하는지 꼭 확인(테스트) 필요
  + 큰거 2개보다 작은거 4개가 낫다
* 백업 및 복구
  + S3를 쓰면 좋음 (Import/Export 기능)
  + Pilot light? : 안쓰는 EC2를 다른 리전에 구축해두고, 데이터볼륨은 적은 볼륨으로 다른 리전에 구축 - 문제시 Route53같은걸로 돌림
  + 완정 동작 저용량 스탠바이 : Route53에서 %로 분배하도록(LBR?) 구성, 스탠바이서버에 저용량으로 동일하게 동작하게 구성하여 본서버가 터져도 약간 버티는 동안 용량을 확장(껏다 켜도 되고 AutoScaling해도 되고... 등등) : 평소에도 체크해봐야 함
  + 다중사이트 액티브 : 돈이 많이 들지만(Utilization은 50%를 넘지 못함) 100%안전하므로 크리티컬할때 

* Amazon Workspace를 통해 관리 가능
* 아마존 복구의 장점: 저렴하게 테스트 가능
* 점진적으로 개선하세요: 백업 > 복구 > 파일럿 라이트 > ...

* HA 데이터 베이스 패턴
  + M - S구성, 추가로 Read Replica를 구성하면 좋다.
    - S를 M으로 변경 가능하고(동일 DNS), Read Replica도 S나 M으로 승격 가능
    - HA proxy를 쓸 수도 있지만, 보통 M/S는 endpoints가 동일하지만, Read Replica는 별도의 endpoint를 갖고 있음
    - R-R는 모두 endpoint가 다르므로 앞단에 Route53을 써야함
  + RDS는 IP가 자꾸 바뀌므로 반드시 DNS 를 쓰세요.

## 가격 고려
* 예약 요금을 쓰면 좋다
* 예약된 인스턴스를 마켓플레이스에서 팔 수도 있고 살 수도 있음(좀 더 싸게 가능)
* 탄력적으로 운영하라
* 스팟도 적절히 사용하면 좋다
* 계정을 분리하고 한명에게 몰아서 결제가능 : 계정모으면 통합해서 청구되므로 할인도 가능
  - 예약 인스턴스를 공유할 수 있음

## 정리
교재 중심으로 공부하면 됨  
살짝 기본적인 아키텍처  
  * Route53 -> CloudFront -> IGW -> VPC -> ...


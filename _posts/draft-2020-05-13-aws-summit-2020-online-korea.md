---
layout: post
title:  "AWS Summit 2020 Online Korea"
date:   2020-05-13 09:00:00 +0900
categories: AWS Summit
comments: true
---
# 신종 코로나 바이러스 19 때문에 온라인으로 개최

## 기조 연설(Dr. Werner Vogels)
* 지금은 기반을 다지기 좋은 때, 변화에 대응하기 위해선 기반이 중요
  + Builder's library:
  + Well-Architected framework: 단순 문서 x, tool로서 아키텍처를 진단해줌
  + 기업은 그간 부족했던 기반을 다지고, 개인은 신기술을 배우세요
* 재택으로 디지털 수요 폭증, 확장성 필요
  + Auto-scaling, Load-balancer, Cloud-front(+S3), Routt53: 근본적 개념 중요
* 미디어 유통에서도 클라우드의 역할이 아주 큼
  + PB급 데이터 전송, 처리, 저장, 보안
  + 진정한 협업을 위한 미디어 Asset 관리(Content lake)
    - AWS Snowball - S3 - EC2 - 보안
* 각 비즈니스는 다른듯 하지만 비슷하고 상호의존적
  + 기술 + 문화 -> 같은 목표로 노력
* 의료서비스 Care Connect
    - Fargate - ECS - Aurora: 확장성 성능
* 빠른 아이디어 실현도 중요
  + Nextdoor (이웃 소통 서비스): 영감을 AWS를 통해 빠르게 구축
  + Filaindiana (지역 슈퍼마켓 이용 효율화): 30시간만에 구축
  + Doyourpart (코로나 교육 및 에방 운동)
* AWS는 다양한 기술과 도구를 제공합니다. 배우고 만드세요!

* 한국사례
  + SKT
    - AWS와 함께 5G MEC구축 - 기지국을 Edge로 활용
    - AWS WaveLength, OutPost와 결합, Edge개발환경 오픈예정
    - 스마트 팩토리, 클라우드 로봇, 산업용 AR, 디지털 헬스케어, 등..
  + beNX
    - BigHit Entertainment에서 만든 가입자 700만명 플랫폼 회사(weverse)
    - 팬덤 현장 구매를 온라인 판매-현장 수령으로 변경
    - Technical challenges: 순간 트래픽 해결, 대용량 푸시
    - 유연성, 자동화, 데이터 고도화
    - 두가지 spike 패턴: 팬의 reaction 대응 / 상품 판매시 요청처리

## 이벤트 드리븐 아키텍처 구축을 위한 적절한 어플리케이션 통합 서비스 선택 및 사용방법(김성진, Solutions Architect)
* 이벤트 드리븐 아키텍처의 개념
  + eCommerce에 주로 사용 - 구매 버튼 클릭 후 진행되는 작업들
    - 동기적으로 처리시 하나의 서비스 장애시 장애의 전파, 서비스 변경, 추가 시 변경 전파
    - 고가용성 이벤트 스트림 이용, 이벤트 생성 서비스는 소비 서비스를 고려할 필요 없음
  + 이벤트란?
    - 상태 변경이 이벤트로 간주 될 수 있음, (주체의)명령이 아닌 (대응자의)관찰
    - 발생한 사건을 표현(이벤트만으로 정보를 충분히 알 수 있어야 함), 중요한 정보, 불변, 과거 시제의 동사
    - 소비 시스템에 대해 생성 시스템은 관여하지 않음
    - CloudEvent라는 표준화 작업 중 (CNCF의 인큐베이트)
    - Blob vs Structured object: 대부분의 장애는 소비 시스템 - json등 동적으로 변경가능한 구조가 좋음
  + 이벤트 드리븐이 적합한 경우
    - 트랜잭션 정보가 포함되어야 함 (stateless에 적합함)
    - 추가비용 없이 적용 가능한 유용한 정보
    - 강력하게 분리된 마이크로 서비스
    - Pub sub 모델
* AWS를 이용한 어플리케이션 간 통합 서비스
  + Amazon SNS, Amazon EventBridge(작년 reinvent에 발표)
  + Amazon MQ(관리형 Active MQ)
  + Amazon Kinesis, Amazon Managed Streaming for Kafka: 대용량 데이터 처리용
  + Amazon EventBridge
    - 손쉽게 구축, 소스에 AWS외에도 SaaS앱의 데이터도 처리 가능, 타겟은 다양한 AWS 서비스
    - 룰 기반으로 이벤트 버스(간단한 가공, 필터 가능) 구축
    - SQS는 메타데이터만 필터링 가능, EventBridge는 페이로드도 필터링 가능
    - SQS와 대비 차이: EB는 SaaS와 구축할 때 권장됨
  + 메시지 체널
    - 지점간 (queue) : 단일 수신자가 수신, 확장에 용이, 최대 트래픽 지정 가능 (SQS)
    - 게시자-구독자 (topic) : 다중 구독, 내구성 (SNS, EB, MQ)
    - 토픽 - 큐 - 연결 : 팬-아웃 형태 구성, 버퍼 역할
    - QoS : 적어도 한번(Standard Queue), 최대 한번(FIFO Queue?), 정확히 한번
    - 정확히 한 번 받아도, 중복 처리는 필요, 멱등성 가진 방식으로 처리 필요
    - 중요한 경우 손실없어야 함: replay queue, dead letter? queue 
  + 메시지 라우팅
    - 토픽 기반: 게시자가 구독자를 고려하여 선택적으로 토픽 - 종속성 발생
    - 메시지 필터: 구독자에 의해 제어, 게시자는 관여 안함, 구독자는 필터링 불필요, 확장성 비용효율
  + 분산 / 수집 구조
    - 견적 시스템의 예, 견적 요청서 분배, 입찰자의 견적서 수집
* 결론: 이벤트 드리븐은 느슨한 결합, 적절한 서비스 및 패턴 선택, 버퍼역할로 속도차 극복, 큐 제약에 대해 파악 잘하고 서버리스 추천 

## Graviton processor instance 살펴보기 (김종선, Solutions Architect)
* 다양한 컴퓨팅 소개와 발전 사항 안내
* 대부분의 서비스는 컴퓨팅으로 시작, AWS에서는 다양한 컴퓨팅을 제공, 22개 리전, 69개 가용영역, 최대규모의 네트워크 인프라
* 2017년 이후 NITRO 아키텍처 적용하여 가볍고 고성능의 보안 강화된 환경으로 구축, 2019년 발표된 NITRO 기반 인스턴스는 성능이 훨씬 좋음
* 프로세서 다양성: Inten, AMD, nvidia, 그리고 Graviton
  + A1 인스턴스: 스케일 아웃에 최적화, 최대 45% 비용 절감
  + 포괄적 에코시스템(다양한 운영체제, 컨테이너) 지원, AWS시스템과 통합을 충분히 지원
* 밴치마킹 : 좋음
* 신규 칩 개발은 신중해야하고 철저한 검증 필요
* 방송 끊김...

## 모두를 위한 하이브리드 클라우드 아키텍처
* 클라우드는 더 이상 선택이 아님, 얼마나 할 것인지만 남음
* 온프레미스는 자원이 한정적임, 클라우드를 통해 
  + 버스팅 수행 가능
  + 백업 및 DR로 구현
  + 애플리케이션 마이그레이션
  + 글로벌 확장
* 전통 비즈니스(통신, 금융, 제조업, 컨텐츠/게이밍), 로우 레이턴시일 경우에는 클라우드 전환 부담
  + 하이브리드 서비스로 커버 가능
* 하이브리드 네트워크
  + AWS Direct Connect
  + Site-to-Site VPN (IPSec VPN으로 온프레미스와 연결)
  + Route53 Resolver (DC또는 VPN연결된 서비스를 DNS로 연결)
  + Client VPN
* 하이브리드 스토리지
  + S3 Acceleration, Kinesis data Firehose, SFTP, Data sync
  + Snowball (데이터를 S3로 오프라인으로 이동)
  + Snowball Edge, Storage gateway
* Storage Gateway: File(S3), Volume(EBS), Tape gateway(S3,Glacier)
  + 게이트웨이에 저장 시 네트워크 통해 AWS에 저장 
* DataSync와 Storage Gateway 시나리오
  + 마이그레이션, 지속적 복제는 DataSync agent활용
  + File gateway는 빠른 전송
  + 상용 온프레미스 백업 서비스의 저장소를 DataSync로 활용 가능
* VMware Cloud on AWS
  + 전통적인 VMware관리를 그대로 사용 가능 
  + 클라우드로 마이그레이션, 데이터센터 확장, 재해복구, AWS와 엮기로 활용
* Outpost
  + AWS 인프라를 온프레미스에 설치/운영, 모든 API/관리체계가 동일
  + 장치 주문, 요구사항 확인 필요
  + 2가지 활용: Native AWS활용, VMware Cloud on AWS활용 
  + Outpost는 가용영역으로 간주됨 

## 최신 컨테이너 기술 소개 및 가시성 향상을 위한 컨테이너 로깅 / 분석 최적화 (유재석/안효빈, Solutions Architect)
* 다양한 컨테이너 제품: ECS, EKS, AWS Fargate(서버리스 컨테이너 컴퓨팅 서비스) 최신 업데이트 소개(유재석)
  + AWS의 컨테이너 서비스 역사
    - EKS: K8s를 관리형으로 제공, 
    - ECS(2015년): EC2는 고객의 VPC, 컨트롤은 AWS
    - ECS에서 Fargate(2017년)지원 
    - K8s는 모든 컴포넌트를 직접 설치하여 사용
    - EKS(2018년 출시) - 컨트롤 플레인만 관리형
    - EKS Phase2(2019년 후반) - 데이터 플레인도 관리형으로 출시
    - EKS가 Fargate지원(2019년 12월)
  + Fargate가 있으면 워커노드를 직접 설치 관리 불필요, 네트워크 스케일링 등만 정의하면 됨
    - ECS, EKS는 오케스트레이션 레이어
    - EC2, Fargate로 컴퓨팅 레이어 구성
    - 보안을 위해 컨테이너마다 VM할당
    - 인프라 관리 바용/시간 축소 (EC2는 고객의 책임 영역)
    - 기존 컨테이너 그대로 배포가능, 사용한 만큼만 비용, AWS서비스들과 연계
  + ECS
    - ECS 클러스터: 논리적 네임스페이스  
    - ECS 작업: 최소 수행단위(K8s의 pod)
    - ECS 서비스: 여러 작업을 관리, 실패한 작업을 자동으로 대체(K8s의 deployment)
    - 클러스터 생성 - ASG 생성 - 인스턴스 시작 - 작업 수행
      - ECS는 CloudWatch 지표 반영, 스케일링 하여 EC2 증감 - 인프라 기반 자원확충은 부정확, 어플리케이션 우선 주의 필요
    - 클러스터 생성 - ASG 생성 - 용량공급자 생성(2019년 발표) - 작업 수행
      - 작업 수행시 자원 부족 - 용량공급자에게 요청 - EC2생성 - 작업배치
      - Fargate용 용량공급자도 존재
    - 용량공급자를 이용하여 다양한 전략 가능
      - 2개의 용량공급자로 가중치를 두고 비례하여 용량공급자가 spot과 ondemand로 운영 가능
      - 가용영역의 불균형(오래 운영할 수록 생길 수 있음) 해소 - 용량공급자를 AZ별로 생성
* 가시성 향상 컨테이너 로깅/분석 최적화(안효빈)
  + 지표(metric): 특정 순간에 발생한 일, 시간에 따른 변화
  + 로그: 불규칙적으로 생성, 원인파악에 도움
  + 추적(trace): 전체 구성요소에서 수명주기 동안 단일 사용자의 요청이 어떻게 되었는지 확인가능
  + 관측(observability): 내부상태를 얼마나 빠르게 측정하는지, 작동여부(모니터링)가 아니라 작동하지 않는 이유를 파악 가능
    - 컨테이너 스택의 계층별 가시성
      - 컴퓨트 서비스(클러스터, 인스턴스)
      - 컨테이너 서비스(서비스, 테스크, 팟)
      - 애플리케이션(각 서비스, 서비스 간)
    - 기존 모니터링 도구와 호환
    - 모든 컨테이너 환경 대응가능한 일관된 도구
  + AWS App Mesh
    - 서비스 매쉬: 서비스간 통신을 관리, 각 서비스별 다른 언어 프로토콜을 일종의 프록시 덮어 gate로 활용
      - 프록시가 직접 서비스와 통신하기도 함
    - 오픈소스 프록시인 Envoy기반 구현
    - 마이크로 서비스 연결 방법을 모델링하여 구성 정보를 계산, 각 마이크로 서비스에 전송 - 가시성 확보, 트래픽을 제어 가능
    - AWS의 모든 컨테이너 환경을 지원
    - 기존 솔루션과 통합 가능, Prometeus, Grafana, CloudWatch, DataDog, Fluentd
    - 일관적 가시성 확보, 하나로 agregate해줌
    - 관측 가능성 향상
  + FireLens: 로그에 특화, 일종의 로그 라우터
    - 컨테이너 런타임 빌트인이 아닌 별도의 이미지로 컴퓨팅과 독립적
    - Fluentd, Fluent bit과 연계
  + Container Insight (CloudWatch, 모든 컨테이너 환경에 대응)
    - Prometeus등이 선호 됨
    - 완전 관리형 관측 서비스, 사전정의된 대시보드 제공, 성능로그 제공
* 활용
  + 인프라 레벨에서 디버깅 뿐 아니라 서비스 레벨, 애플리케이션 레벨까지 디버깅 

## AWS 관리형 서비스를 활용하여 Kubernetes 를 위한 DevOps 환경 구축하기 (김광영, Solutions Architect)
* 속도와 민첩성이 중요 - MSA로 바꾸고 Serverless로 그리고 CICD 필요 -> DevOps
  + DevOps: 문화, 방법론, 도구
  + 적은 인력으로 적용하려면? IaC화 필요
  + Application Layer도 IaC화 가능하다면? Docker + Kubernetes 부각
  + 모든 계층을 추상화 한 K8s를 prod에 적용하기 쉽지 않음
    - 보안, 복잡성, 모니터링, 네트워킹, 스토리지, 로깅, 스케일링, 안정성 
* EKS의 클러스터 관리
  + EKS는 오픈소스 (수정없이)그대로를 구현, CNCF인증
  + EKS는 AWS가 클러스터를 관리해줌
    - Phase 1: EC2를 클러스터로 활용, 컨트롤 플레인 지원, Master는 2중화 Etcd는 3중화 
    - Phase 2(작년 출시): 관리형 워커노드, 별도로 EC2를 노드로 할당할 필요가 없음, EKS가 autoscaling
    - AWS Fargate for Amazon EKS 출시: 
      - 워커노드, 오토스케일러, 미사용 유효자원 관리 불필요
      - 아직 미지원 부분도 존재
    - Fargate와 믹스로 사용 가능
* AWS관리형 서비스로 CICD 구축
  + 포괄적인 엔터프라이즈급 툴셋 제공: Cloud9, CodePipeline, CodeBuild, ECR
    - CodePipeline: 지속적 전달 서비스, 젠킨스와 통합 가능
    - CodeCommit: 소스 제어 서비스, Git기반 
    - CodeBuild: 컴파일, 테스트, 패키징, 자동으로 slave 확장되면서 실행, Docker로 빌드환경 제공
    - ECR: 확장가능, 고가용성, 고내구성, 암호화, IAM으로 접근제어, Immutable image tag 지원, Image scanning 
  + Cloud9 에서 코드 커밋, 코드 파이프라인이 코드 빌드 후 ECR배포, EKS가 image를 이용해 팟 배포  
* 모니터링
  + 복잡한 구조로 모니터링이 어려워짐 
    - 메트릭(CloudWatch Metrics, Container Insight), 로그(CloudWatch Logs, Log Insight), 트레이스(X-Ray) 3가지 구성요소 
    - Container Insight: 컨테이너의 컴퓨팅 성능을 모니터링
    - Log Insight: 대화형으로 로그 분석
    - X-ray: 성능 분석, 병목 확인
  + 컨테이너 모니터링
    - AWS관리형 서비스로 대부분 모니터링 가능
    - 다양한 써드파티 업체와 연계도 가능  
* AWS가 가장 선호됨
  + AWS와 긴밀한 통합
  + DevOps 워크플로우
  + 보안 및 규정 준수 
  + 구성은 AWS에 맡기고 비즈니스에 집중

## 회사 계정/패스워드 그대로 AWS 관리 콘솔 및 EC2 인스턴스 사용하기(이정훈, Solutions Architect)
* IAM 101
  + User와 Role: password가 있냐 없냐로 구분할 수 있음, Role은 자격증명 된 누구에게나 줄 수 있음 
  + Role을 통해 account 너머에도 권한을 줄 수 있음
  + 클라우드 시대에는 IAM관리가 가장 중요해 짐
  + AWS SIGv4로 재현(replay) 공격등에 대응
  + 모든 이력이 CloudTrail에 남음
* EC2 인스턴스 계정 통합
  + ec2-user를 그대로 쓰거나 별도의 user를 쓰거나..
  + Amazon EC2 Instance Connect
    - 2019년 출시: WebBrowser로 접근, 또한 CLI, SSH도 지원, 
    - 60초 유효한 임시 SSH키(관리 불필요, IAM으로 제공), CloudTrail로 기록
    - 리눅스만 지원 (AWS Linux가 아니면 agent필요)
    - 흐름: 운영자가 EC2 Connect Service에 공개키 전송, 메타데이터를 이용하여 EC2 인스턴스에 투입, 임시 개인키로 SSH 연결
    - IAM유저의 권한에 EC2InstanceConnect:SendSSHPublicKey권한을 부여하고, Condition에 "ec2:osuser": "${aws:username}" 추가 필요
    - Browser가 아니라 사용자 console로 접근하기 위해서는 aws ec2-instance-connect 로 기존에 사용중인 키를 제공하거나 임시 발급하고, 그 키를 이용하여 인스턴스에 접근 (조금 불편...)
    - mssh라는 별도의 CLI를 제공 ($ mssh username@i-instanceId)
    - Role 기반으로 사용 가능
  + AWS Systems Manager Capabilities - Session Manager
    - Instance Connect 기능을 대부분 지원
    - 22번 포트를 안열어도 됨
    - IAM user의 tag에 SSMSessionRunAs로 osuser를 등록해줘야함
    - 모든 이력이 CloudWatch logs에 남음, 추적성 확보
* 계정관리는 없애고, 접근 제어는 그대로
  + IAM이 아니라 회사 계정으로 사용하고자 할때
  + 아이덴티티 페더레이션을 사용하면 한 ID 만으로, 민감한 정보는 한 곳에 저장하며, 내부 통제와 통합되어 일관된 장점이 생김
  + SAML2.0: 사용자가 기업 데이터센터네 로그인을 하면 SAML인증서를 받게되고, 이를 통해 AWS 콘솔에 로긴
  + 유저를 기업 AD에 추가, AWS 로그인 시 role선택하여 접속
  + SAML문서에 해당 유저의 ARN role number가 기록되어 있음 (즉, AWS role을 먼저 만들고 SAML에 가용한 role을 정의해두어야 함)
  + 수 많은 프로젝트, 리소스.. policy 관리의 부담 -> Attribute based access: AWS에서는 tag
    - Session tags
  + AWS Single Sign-On: 수 많은 계정을 쉽게 관리하기 위함
    - SAML로 접속해 여러 계정을(Organization 기반으로) 접속하거나 권한을 줄 수 있음(좀 헷갈림)
  + AWS Organizations: AWS 계정을 조직 구조에 맞추어 정의
    - 리소스 공유, 빌링 통합, 서비스 접근 권한 통제
    - Tag기반 정책
* 결론
  + 사용자 저장소는 최소화
  + 인스턴스 접근을 IAM 서비스와 통합 고려
  + SAML이 있다면 SSO와 연동
  + 아이덴티티 관리 통합으로 편의성, 보안, 추적성 향상
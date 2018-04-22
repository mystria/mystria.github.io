---
layout: post
title:  "AWS Summit 2018 Seoul - Day 2"
date:   2018-04-19 09:00:00 +0900
categories: AWS Summit2018
comments: true
---
## 마이크로 서비스 Spinnaker
  * 대기업의 DevOps
    + Dev가 불을 지르지 않도록 Ops가 관여해서 개발 진행
    + Ops가 Infra를 코드로 관리 -> Immutable Infrastructure
      - IaC(Terraform), CD(Spinnaker)
      - 예전에는 CloudFormation과 Chef를 활용: Milk과제
  * 대기업에서는 어떻게 했는가?
    + Run Book: Incident 발생시 대처 매뉴얼 작성
      - VictorOps로 alert처리: 일단 재부팅(ImmutableInfra), ticket생성 후 재현, 서버 교체
    + 이러한 절차를 정착시키기 위해...
      - Immutable Server(phoenix server), IaC
      - Spinnaker: 손쉬운 구성, 배포 모델(전략)을 지원
      - 다양한 도구들: Jenkins, nGrinder, Locust, Datadog, Zipkin, Sumologic, Slack, Jira
  * Immutable ...
    + 수리하는게 아니고 새제품으로 교환
    + Pakcker를 활용하여 서버 이미지, 또는 컨테이너 이미지 생성
      - AMI는 몇개월에 한번씩 업데이트 -> 애플리케이션 이미지를 얹어서 출시
  * IaC
    + 시뮬레이션, 보안 (검토 후 동일하게 적용가능), 원클릭 DR
    + 휴먼 에러 방지
    + Terraform의 Module registry활용
  * CD
    + Pipeline: 테스트 자동화, 테스트 통과 실패시 절대 차단!
    + 배포 모델을 적용한 안전한 배포
    + 운영자들도 보기 쉬워야..
    + 커밋 스테이지 -> 액샙턴스 테스트 -> 카파시티 테스트 -> 매뉴얼 테스트 -> 배포
  * Spinnaker
    + 다양한 퍼블릭 클라우드 지원
    + AMI baking은 Jenkins에서 trigger활용 Packer -> Spinnaker로 트리거
    + 강력한 파이프라인, 배포, 시각화 도구(OAuth도 지원), 1.6.1이후 버전

# 기조 연설
  * Swami Sivasubramanian
    + ML활용
      - 로봇으로 Amazon 물류관리 - 창고에서 물건을 찾아와 포장
      - Amazon 추천
      - Kindle에서 x-ray(책에서 주인공이 언제 얼마나 나오는지)
      - Amazon Video에서 화면에 나오는 배우들이 누군지 알려줌
      - Alexa(Echo): 자연어의 맥락을 구분
  * 모든 머신러닝 기술을 학생과 과학자에게 제공
    + Data Scientist: Framework infra
      - P3인스턴스(가장 파워풀), DeepLearning AMI(환경구성완료)무료 제공
      - Gluon: 확장성(텐서플로 CNTK등), 유연성(파이토치) 둘 다 잡게 지원
    + Engineer: Platform - Amazon Machine Learning
      - 개발자들이 쉽게 머신러닝울 사용하게
      - Amazon Mechanical Turk: 필요한 신뢰할 수 있는 데이터셋 제공(처음엔 사람이 주석을 달아야 함)
      - Amazon SageMaker: 원클릭 학습/배포, 원하는 프레임워크 선택, 초당 과금, 간단하고빠른 훈련, 대용량 고신뢰 지원, 다양한 알고리즘 지원
      - AWS DeepLens: 딥러닝 지원되는 개발자용 비디오카메라 > AWS로 스트리밍, 세이지메이커로 분석: 미술양식변환, 사물인식, 개체 탐지 등의 템플릿 제공
    + 모두: Application(서비스) 출시
      - Polly: 다양한 언어와 톤의 목소리 제공
      - Rekognition: 딥러닝 기반 이미지 분석(객체탐지, 얼굴분석/인식/실시간(0.5초이하)식별, 텍스트인식)
      - Rekognition Video: 동영상분석(전후맥락감지, 사람추적, 사물인식 s3와 키내시스활용)
      - Amazon Transcribe: 딥러닝 기반 자동음성인식(스피치 투 텍스트-구두법, 다중화자인식) 콜센터/자막/텍스트기록에 활용
      - Lex: 알렉사의 중추(아마존 서비스와 연계 또는 활용)이자 봇 생성에 적합
      - Amazon Translate: 뉴럴 딥러닝 기반 번역(12가지 언어, 언어인식 및 실시간 번역),
      - Amazon Comprehend: 딥러닝 기반 자연어 처리(감성, 주요엔터티, 핵심문구, 언어 파악-고객 리뷰 파악 및 문서 분류에 도움)
  * 기계학습과정
    + 데이터 추출. 선별 포멧 변경. 데이터 모음 준비 데이터 변형. 모델 훈련. 모델 평가. 프로덕션 배포. 모니터링 디버깅 리프레쉬. 처음부터 반복
    + SageMaker 과정
      - 구축. 훈련. 배포
      - 노트북(기 구성된 인스턴스)

  * 엔터프라이즈에서 아마존 경험을 활용
    + 아마존 ML 솔루션 랩: AWS의 경험과 지식 전수
    + ML 리서치 grants: 대학과 연구 연계
    + 기업들과 협력,
    + AI startup challenge

  * IGAWorks
    + 요구사항: 마케팅 분석 솔루션 제공 - 글로벌 규모의 요청을 처리, 빠른 머신러닝 모델 구축
    + 아마존 워크로드 활용 + SageMaker: 적은 인원(석박사10여명)으로 빠른 구축 가능(프로세스 단순화, 최신 알고리즘, 인프라확장성), 완전관리형 서비스

  * 조선 미디어 그룹
    + 요구사항: 디지털 윤전기(새로운 고객 전달) - 조선오디오출시, 자체 컨텐츠(신구 데이터)를 분석하여 디지털로 공급
    + 클라우드 도입, 아마존폴리를 통한 오디오 서비스

  * 현대카드
    + 요구사항: 빅데이터를 이용한 고객 분석, 데이터 디지털화, 카드사용분석 후 해외 패션사이트, 취향, 트랜드 분석 후 추천
    + 아마존 기반 디지털 파운데이션 구축, AI/ML활용

## DR
재해복구
  * 여러가지 이유로 IDC가 날아갈 수 있음
    + 자연 재해, 사고, 관리자 실수 등
  * 전통적 DR접근
    + RPO, RTO: 엔터프라이즈 에선 아직 유효, 이미 구축된걸 바꿀 필요 없음
    + 그러나 새로운 서비스에서 굳이 할 필요는 없음
  * 구현 종류
    + IDC를 여러개, IDC와 Public Cloud, Public Cloud를 여러 종류, Region을 분리, AZ를 분리: 앞쪽이 더 강력하지만 뒤쪽이 효율적
    + 게임에서는 규제에 따른 필요성은 없지만, 어카운트/과금에 대해서는 전통적 DR고민
    + 흔한 방법 - 사이트 복사: 사용여부를 떠나 똑같은 복제 -> 낭비(Active에서 StandBy로 정기적으로 데이터 동기화), 비용과 노력은 2배 효율은 오히려 떨어짐
    + AWS DR: 사용자 경험 개선, AWS리소스 효율적인 활용, 사용자 지역장애에 대응 가능
  * DR 고려사항
    + 복구 소요 시간
    + 비용
      - Full, Data only, Data only batched, Small scale
      - 복구 시간도 결국 돈: 돈, 인적 비용, 시간, Data loss
      - 과거관점: 비용 vs 시간 vs 데이터 로스
    + AWS 서비스 활용
      - 금융이 아니라면 조금 더 유연하게..
      - On-Demand Infra, Region간 구성 표준화, Region내 AZ는 동일, Region간 통신은 빠름, AZ to AZ는 더 빠름
      - Multi-AZ: 물리적 격리 충분, 네트워크 분리, 그러나 네트워크 거리(성능)은 아주 가까움(DB싱크 가능), Managed Service는 Multi-AZ 대응
      - 단일 지역/다중 AZ가 기본: AZ단위 장애가 대응됨
      - Managed Service(Aurora, S3, Lambda, Kinesis, SQS 등) 추천: URI는 region단위로 노출되어 AZ 상태와 독립적으로 활용
      - AZ를 사용하길 추천(이유는 AZ간의 빠른 속도와 물리적 분리)
  * GOB(Global One Build)
    + 클라이언트는 앱스토어에서 배포, 스크립트는 CDN(CloudFront)로 다운로드
    + 서버는 DDB(DynamoDB) 추천: 리전간 스트리밍 동기화, Global Table(GT)활용 - Single source of truth
    + 아니면 Aurora MRMM(Multi Region Multi Master)를 추천 - Capacity도 늘이고, 리전간 데이터 통신이 빠름
    + S3를 이용하여 Daisy chain으로 리전 배포

## On-Premise 에서 Cloud 전환기
  * 메가존
  * 고객사 현황과 클라우드 도입 이유
    + Cisco - Apache - Oracle - OracleDB - EMC
    + Monolithic: RHEL - Java - Spring - SVN, Maven, Eclipse
    + Agile + DevOps + Container기반 MicroService로 전환하고자 함
    + 왜 Cloud? Fail fast, Fail early, Learn cheaply - 성공여부 확실하지 않음, 사회공헌 서비스/ 프로토타입 서비스라서 용량산정 불가능
  * To-Be 아키텍처
    + Container기반 서비스, CICD를 위해 CodeSeries와 CloudFormation 적용
    + 아마존 managed service 많이 적용
    + 그러나.. 문제 발생
      - 쓰던 도구에만 익숙해서 새 도구 거부 - 기술적 이슈
      - 개발자가 DevOps나 NoOps는 감당하기 힘듦 - 문화적 이슈
  * 어떻게 도입
    + 순차적 도입
      - 먼저 클라우드 도입 후 마이크로 서비스로 전환
      - 마이크로 서비스로 전환 후 클라우드 도입
      - AWS 도구들도 쉬운거 부터 적용 후 차차 세부적인 도구 사용 (ElasticBeanstalk, CodeStar -> ECS, CodeSeries)
      - 최적화 아키텍처 제안: API Gateway, Cognito, Lambda
  * 레슨런
    + 결국 도구 보다는 사람
    + 팀 단위, 프로젝트 단위로 선도 적용 후 다음 과제에 확산
    + Doing Agile/Being Agile, One at a time/ Not All at once, OpenMind Collaborative
  * DevSecOps
    + 하지만 보안은?
      - CICD를 도입하여 보안이 강화, 하지만 좀 부족
      - 개발 초기부터 보안에 신경쓰자: Security as Code
      - 마지막 배포 전에 보안 적용 말고, 모든 파이프라인 과정에서 적용
      - CICD과정 중간중간 보안 검증, 산출물에 대한 checksum
      - AWS환경에도 서버단에서 보안강화가 필요(기본적으로 다 열려있음)
    + 해야할 것
      - 보안 담당자와 친해지고
      - 보안 상태와 요구사항 이해
      - MFA사용
      - 현재 환경에 대해 보안 검증 계획 수립
      - 등등..

## Enterprise AWS 도입 사례
  * 왜 BSG
    + SAP 제품 파트너 사, AWS 컨설팅 파트너, SAP on AWS
  * Enterprise의 AWS 도입 동기, 도입 시 요구조건
    + 기존 스타트업 주류 에서 2016년 이후 Enterprise도입 증가
      - Enterprise의 경우 규모도 크지만 네트워크/OS/Application/Interface까지 도입해야하는 고민
    + AWS 검토 이유
      - SAP 업그레이드, 기업 구조 변화, 시스템 도입 전 검증, 기존 서버 노후화와 단종, 중국/GDPR규제, DR, 글로벌 진출, IDC 및 SysOps인력 비용 감축
      - 비용 42, System 유연성 효율성 32, 바로 적용되는 즉시성(프로젝트 기간 단축) 15, HA/DR 6, 글로벌확장 5, 정부 규제 1
    + 도입시 요구조건
      - 촉박한 프로젝트 일정, 투자/유지(직접비, 간접비)/서버라이선스 비용 절감
      - 시스템 복제, 테스트 환경
      - 마이그레이션시 다운 타임 최소화
      - AutoScaling, HA, DR 구성
  * 고객 사례
    + SAP 업그레이드
      - 금욜18시~월욜9시 80시간 이내에 적용: EC2 instance 성능을 높여 제한된 시간에 마이그레이션 후 운영시 Scale down
    + 서버 노후화로 교체
      - Unix를 Linux로 다운사이징(Linux신뢰성도 높아짐), 3년 약정 인스턴스로 비용 절감
    + 법인 분리로 SAP시스템 분리
      - 2주 일정, 시스템 인터페이스 유지: H/W준비 기간 0, 인스턴스를 동시 생성하여 병렬 작업, 작업 완료 후 인스턴스 반납, 시스템 인터페이스를 동일하게(똑같은 보안/네트워크 설정) 구축
    + 해외 시스템의 재해로 인한 중단, 현지 인력 부족
      - HA/DR구축, 해외 관리 비용/risk절감: AWS 다양한 리전 활용, AZ기반 DR구축(재해발생 시 활성화 - 서버가 중지상태에선 비용없음)
    + 신규 구축
      - 직접비(H/W) + 간접비(인건비, 전산실 유지, 네트워크): AWS 구축비용 비교, 3년 약정, 버퍼 비용 없음(필요시에 Scale-out), 미래성장/peak대응

## AWS에서 Kubernetes 실전 활용하기
GCP에서는 잘됨, AWS에서는 여러 도구가 필요
  * 왜 Container
    + 서버의 상태, 관계, 히스토리를 알기 어렵다
      - Docker 가상화는 hypervisor보다 성능 손실이 적음. 서로 간섭하지 않게 고립시킴
      - 가상화를 통해 container화 하면 관계/히스토리를 알 수 없는 문제 해결
      - Docker를 쓰면 명세인 dockerfile을 만들기 때문에 이를 코드화 하여 이력관리(깃 히스토리를 도커이미지의 태그로 지정하여 뭐가 변경되었는지 알 수 있음)
  * 왜 container orchestration
    + Application Load Balancer를 이용하다 Docker도입 시 nginx proxy로 ALB 역할 전환 -> 여전히 인프라 관리 복잡: 컨테이너 오케스트레이션 툴로 이를 극복
  * 왜 Kubernetes
    + Cluster/Node(EC2 instance)/Pod(Container들을 담는 그릇)
    + RC(Replication Controller: pod control, 원래이름은 ReplicaSet)/Deployment(업데이트, 배포 모델 관리)/Service(외부 배포)
    + Automatic binpacking
      - 가방을 최적화해서 담는 알고리즘을 이용해서 알아서 pod에 컨테이너 담아줌
    + Horizontal scaling
      - Nod가 엄청 많아지지 않게
    + Automated rollouts and rollbacks
      - ?
    + Self-healing
      - 망가졌다 판단되면 마스터가 자가 복구
    + Service discovery + load balancing
      - 복잡한 설정이 필요하지만 알아서 잘 해줌, Internal: ClusterIP, External: LoadBalancer/NodePort(비추)
  * 왜 kops?
    + AWS의 클러스터 관리를 자동화 해줌(EC2, Route53, S3, IAM, VPC 권한)
    + 고가용성 K8S 마스터 제공(두개 이상의 AZ권장)
    + 기본 dry-run(--yes), 멱등성(항상 동일한 결과)

## 특권 계정 관리(PAM)
  * 공유계정(특권계정) 관리
    + 비인가 접근보호:
    + 활동 모니터링, 감사
    + 권한 상승 제한
  * 하이브리드 환경에서 제한
    + 중앙 접근 통제 솔루션 필요
    + 접근 경로 단일화
    + 사용자는 접근통제솔루션에만 접근
    + 멀티플랫폼 전용 클라이언트 제공(업체 광고)
    + 특정 프로토콜/ 보안 콘솔로 접근
    + 패스워드 관리/정책
    + 기존 소프트웨어 하드코딩 패스워드를 라이브러리로 주기적 갱신
    + 위험 명령어, 경유 접근 차단
    + 클라우드 특화 기능
      - ip변화 scaling 등에 대응
      - ssh키 관리
      - 아마존 콘솔 사용 이력을 cctv처럼 레코딩으로 기록, s3에 저장
      - 아마존 autoscaling에 잘 대응
      - accessKey 관리 및 권한 통제
      - 다양한 클라우드 지원
      - rest api로 외부연계
      - 머신러닝으로 능동적으로 위협 분석
      - 접근 모니터링 대시보드 제공

## e-Cormerce 추천 시스템
  * 추천의 필요성:  추천으로 구매 이어질 확률 70퍼 정도
  * 추천의 종류: 아마존 추천 방식과 넷플릭스 추천은 다름(적극적, 비적극적)
  * 추천(알고리즘)의 종류
    + CF user based: 비슷한 사람 추천, a와 b의 취향
    + CF item based: 연관 상품 추천, a와 b가 같이본 물건
  * CF구현 방법: 메모리 맵, 그래프 분석, 단순 카운팅(구현 쉽고, 추가 작업 가능)
    + 구현방법이 뭐든 스케일문제에 직면: 메모리가 커지거나 관계가 복잡해짐
      - 유사 성향으로 프리 클러스터링하여 쪼갬: MinHash, 사이즈를 줄여 속도 증가, 하지않으면 죽거나 끝나지 않음
    + CF단점: 콜드 스타트(신상품, 롱테일) 그래서 CBF
  * CBF: 컨텐츠 기반?
    + 디스크립션과 타이틀로 유사성 판단, 머신러닝 발전에 힘입음
    + 텍스트 기반: 좋지만 추가 필터(브랜드 등)가 있으면 더 좋음
    + 이미지 기반: 딥러닝 기반 벡터값으로 유사한 이미지 추출 - 패션 쪽에 적합
  * 하이브리드: 같이 쓰면 더 좋다
    + 메인은 CF(효과 좋으나 커버리지 낮음), CBF(커버리지 높음)로 보완
    + 패션, 가구는 CBF로
  * AR(Association Rule)
    + 같이 구매한 상품으로 추천(기저귀&물티슈)
    + 커버리지가 낮음 -> AR로 정확도 높이고 구매로그 기반 CF로 보완
    + 함께 구매하는 제품 추천으로 많이 활용됨
    + 유사도 알고리즘: 후보군 도출 후 줄세우기
      - Jaccard: 교집합/합집합
      - Cosine: CBF는 vector(방향성)이므로 코사인으로 방향유사도 판단
  * 추천 시스템 아키텍처
    + 최근 본 상품: api gateway - lambda - elasticache로 redis의 sorted set으로 저장(자연스레 타임스템프로 저장됨), TTL로 저장기간 정의 가능
    + 데이터 레이크: api gateway - lambda - kinesis firehose - s3 - glue: 사용자 로그를 스트림으로 키네시스 저장, 잘라서 s3저장 글루로 데이터 뷰 제공(s3에 단일 데이터 보관, 뷰 별로 데이터를 중복할 필요 없음)
    + EMR로 추천 쉽게 구현후 ddb에 저장(미리 추천셋을 정해서 저장해 둠, 실시간 추천은 느리니깐) -> SageMaker로 쉽게 대체가능. 추천!(Factorization Machines가 간단한 추천 알고리즘임)
  * 성능평가
    + AB Test: 온라인:??  오프라인: 에러율 체크?
    + MAB: 비교해서 위너에게 90% 몰빵?
  * 정리
    + 추천은 ui에서 알고리즘까지 유기적, 알고리즘만 중요한게 아니고 어디에 노출할지 순서를 어떻게 표현할지도 중요한 팩터
    + CF는 이미 오래된 라이브러리 - 추가 작업을해야 Amazon급 추천이 가능
    + 후처리(유사 이미지, 같은 제품명)로 동일한건 걸러낸거나

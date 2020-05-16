---
layout: post
title:  "AWS Summit 2018 Seoul - Day 1"
date:   2018-04-18 09:00:00 +0900
categories: AWS Summit
comments: true
---
# 기조 연설
  * Dr. Werner Vogels
    + Build On?
    + Monolith -> 21세기 새로운 클라우드 아키텍처의 변화
    + Cloud9 Pipeline 수 많은 Resources, 타 vendor보다 강력한 서비스들
    + Machine Learning: 개인화, 상품추천, 자율주행, AmazonGo
      - 모든 개발자에게 ML 기회를...Stack(tool과 infra)제어 가능 : SageMaker
    + Database: DMS, Aurora, Aurora Serverless(예측 불가능 traffic), DynamoDB
    + Data Lakes: S3 -필요한 데이터만-> S3 Select, Glacier Select
      - Glue
    + Well-Architecture: 운영 우수성, 보안 및 규정 준수, 안정성, 성능 효율성, 비용 최적화
      - 운영 우수성: Game day(AWS를 부숴봄)
      - 보안 Best Practice 제공: 모두가 보안을 책임져야 함(AWS와 상관없더라도), 운영 자동화 추구, Amazon Inspector, CloudTrail, Config Rules를 통해 지속적으로 감시, 암호화 잘 해줌, Certificate Manager제공(사설 인증 서비스 출시), Secret Manager 출시(비밀번호 관리)
    + Compute:
      - 가상서버: 다양한 종류의 Instance type제공
      - Container: 마이크로 서비스에 적합(작은 서비스 단위로 scaling, 분산으로 보안강화), ECS, EKS(Kubernetes), Fargate(클러스터 관리 없이, 컨테이너 단위로 개발 집중)
      - Serverless: 미래는 여기, 비즈니스 로직에 집중 - 유연성, 저비용, 고가용성

  * LG전자
    + 요구사항: Connected, Collaborative, Evolving
    + 왜 AWS를 사용하는가? Scalable, Managed service, Serverless
    + AWS기반 자체 domain특화 DB 시스템 구축
  * AmorePacific
    + 요구사항: Mobile First, Cloud Native, Agile
    + AWS의 대용량 storage활용
  * 신한금융그룹
    + 요구사항: 고객경험강화, 운영효율화, 글로벌 확산
    + AWS로 개발 속도 1.5배, AWS와 같은 Public Cloud도입 - 2020년 Cloud로 migration중, 규제대비 사내망과 hybrid, 비중요/비금융 시스템 먼저 적용

## 모놀리스에서 마이크로서비스 아키텍처로의 전환 전략
어떤 작업을 해야 마이크로서비스를 전환할 수 있을까?
  * 마이크로서비스란?
    + 강결합으로 만들어진 모노리식 vs
    + 장점: 장애에 강함, 장기적 기술 종속 감소, CICD 장점 극대화
    + 단점: 개발환경 미성숙, 배포의 복잡성, 시스템 메모리 소요 증가
    + 스타트업 관점: 빠른 성장에 개발시간 많이 소요, 처음부터 마이크로서비스 하기 어려움
    + 엔터프라이즈 관점:
    + 기능 명세를 작성하여 AWS서비스와 매핑
  * 마이크로서비스의 발견
    + 안정적인 아키텍처, 연계가 강한(cohesive) 기능의 집합, 같은 이유로 변경되면 하나의 서비스, 서비스간에 약결합, 각 서비스는 테스트 가능, 2 pizza team
    + 분할 기준
      - 비즈니스 역량: 비즈니스 목적, 가치 창출 기준
      - 서브 도메인: Domain Driven Design, Core/Supporting/Generic 구분, 특정하기가 어려움(이상은 좋으나..)
      - 실무적인 접근: API로 정의, API기준으로 모듈 정의, 코어와 아닌걸로 구분, Database로 분석(어느 서비스에서 어떤 데이터를 쓰는가)
  * 마이크로서비스 아키텍처 패턴(발견, 패턴, 분할, 매핑)
    + 데이터베이스 패턴: 서비스별 테이블, 서비스별 스키나, 서비스별 데이터베이스
      - 외래키 관계 제거, 공유 데이터를 공유 서비스화, 스키마 분리, DB 리펙토링
    + API GW패턴: 다양한 app의 단일한 접근 포인트 제공
    + 회로차단기 패턴: a->b->c 일 경우, 프록시를 통해 서비스 호출
    + 등등...
    + 고민할 것: 확장성, 가용성, 민첩성, 운용편의성, 성능 고려
  * 컨테이너와 AWS서비스
    + 컨테이너 고려
      - 마이크로 서비스 아키텍처의 고려사항과 동일
      - 어플리케이션에서 마이트로 서비스, 자체 SLA충족
    + 다른 애플리케이션 스택, 다른 하드웨어 배포 환경, 다른환경을 운용할 방법, 다른환경으로 이전할 방법
      - 배포 단위, 가볍고 이식성 좋고 일관적, 어디서나 실행가능, 무엇이든 실행가능
    + 2세대 기술: EKS, Fargate, Lambda, Step Functions, API Gateway 등으로 구성
  * 결론
    + 초기 시스템 구성시 부터 고려
    + 기존 앱 전환 계획
    + 발견,패턴,분할,매핑 단계적 접근
    + 단위 구분, 요구사항 고려, 데이터 분할에 대한 전략
    + 컨테이너 및 서버리스 활용
    + AWS 서비스와 매핑
    + 지속적 진화(개선)

## AWS 기반 Kubernetes 정복하기
  * AWS Docker 컨테이너 환경
    + ECS, Fargate: VPC, AWS와 긴밀한 통합
    + ElasticBeanstalk: 프로젝트 단위로 인프라 자동관리
  * AWS 배포 환경
    + ?
  * EKS 소개
    + K8S를 그대로 사용가능하게 서비스 제공
      - 엔터프라이즈 기업이 프로덕션 등급으로 사용할 수 있게
      - 네이티브 및 업스트림 K8S 경험을 제공
      - 다른 AWS서비스(ELB, S3, IAM 등)와 간편하게 연동
      - K8S 발전에 기여
    + 가용영역 3개에 마스터노드, Etcd, Instances를 자동으로 배포
      - 마스터노드와 Etcd가 AWS관리형 서비스로 포함되어, 단지 인스턴스(워커)만 관리하면 됨
      - kubectl -> mycluster.eks.amazonaws.com : Cloud9를 이용하여 개발자에게 Kubectl권한 통제 가능
      - Fargate를 쓰면 인스턴스 관리도 불필요
  * EKS의 여전히 남은 문제점
    + 네트워크, 스토리지, 보안, 컴플렉서티, 로깅, 오케이스트레이션, 모니터링(cncf 조사결과)
  * EKS Network Option
    + SG설정은? 방화벽은? 컨테이너/포드/서비스 단위 라이팅은?(내부 시스템간의 보안도 중요), 컨테이너 제거시 연결 설정등은 어떻게 설정?
    + Container Network Interface(CNI)
      - CNI플러긴을 사용한 Native VPC네트워킹, ENI를 CNI가 대체
      - Tigera의 Calico(OpenSource) 참고
      - AWS의 Shield, WAF도 사용가능
    + 멀티 네트워크: 다중 VPC지원(Network LB와 TLS활용)
  * EKS Storage
    + Stateless한 architecture라고 하더라도, storage 입장에서는 구현 어려움
    + 개발자 역량은 애플리케이션 구현에 투입
    + 클라우드 네이티브 스토리지 특성: 애플리케이션 위한 구조, 다양한 보안옵션, 플랫폼 독립적, 애자일, 동적 구성, 충분한 성능, API호출 가능, 지속적 고가용성
    + Container Storage Interface(CSI) 제공
    결론이 뭐야?
  * 보안
    + EKS는 PrivateLink를 이용해 AWS자원과 연결됨
    + DirectConnection으로 온프래미스 컨테이너들(?)과 연결
    + IAM인증을 K8S 업스트림이 지원
  * 로깅
    + 네트워크 플로우, 액세스 로그, 사용량
    + 애플리케이션 이벤트 로그, 사용자 로그
      - 마스터노드는 CloudTrail, CloudWatch로 연동
      - 컨테이너는 힙스터(그라파다, 등), 헬름(프로메테우스) 구성으로 모니터링 가능 with Cloud9
      - 애플리케이션 로그: CloudWatch 도커 에이전트로 지원
  * 새로운 소식
    + K8S적합성 인증: cncf의 K8S 업스트림과 동일함을 인증
    + 서비스 디스커버리: Route53을 이용해 서비스 검색
  * 결론
    + EKS은 오픈소스 업스트림과 동일
    + 매니지드 서비스
    + CNI, CSI등 통한 운영 효율
    + 기존 AWS와 완벽한 통합
    + 다양한 네트워크 워커 노드들과 연동
    + 오픈소스 플러그인 그대로 지원
    + Shield, WAF, RedShift, Athena 연동

## Github to Lambda
  * AWS CodeStar: 코드 등록, 빌드, 디플로이
    + Github -> AWS CodeStar(GitHub의 변경사항을 notification받음) -> 변화내용을 적용
    + IAM User 생성
    + AWS CodeStar의 Project생성(User이름으로)
    + GitHub에 적용 후 Delivery 확인
    + CodeStar연계시 자동으로 Repository등도 생성되고, Code Pipeline도 구성됨
      - Code커밋 시 파이프라인 동작 후 디플로이 됨
  * GitHub 소개
    + ldap, saml과 연동됨
    + branching
      - protected branches
    + Pull request
    + Code review
    + Publish your app as Open Source
    + Collaborate with others

## 고급 서버리스 배포
  * 서버리스 -> 배포 -> 운영(테스트, 모니터링, 로깅, 문제해결)
  * 서버리스
    + 유휴상태 비용 없음, 프로비져닝 할 서버 없음, 사용량에 따라 확장, 가용성/내결함성 내장
  * 서버리스 구현
    + 함수 준비(Node.js, Python, Java 등)
    + 람다에 배포
    + 이벤트 소스(trigger): 함수를 호출함
    + 금일은 배포와 호출 부분
  * Lambda 실행 모델
    + 동기(Push): API Gateway같이 외부의 호출
    + 비동기(Event): SNS 또는 S3의 이벤트를 detecting
    + 스트림 기반: dynamoDB, Kinesis 등을 지속적으로 polling
  * 서버리스 활용 사례
    + 웹앱, 백엔드, 데이터 처리, 챗봇, 등등..
  * Pipeline
    + 소스 -> 빌드 -> 테스트 -> 프로덕션
    + CICD: CodeCommit, CodeBuild, CodeDeploy -- CodePipeline >> CodeStar(Project화, 자동 구성)
  * 새버전 배포 방법
    + 고려사항: 사용자 영향 최소화, 롤백, 실행 모델 요소, 배포 속도
    + 배포 패턴: All-at-once, Blue-Green, Canary(잘 안되면 롤백)/Linear(점진적으로 새버전으로 이동)
    + 도구들: CloudFormation, SAM(Serverless Application Model: 서버리스 최적화)
      - SAM으로 간단하게
    + 고급기술
      - 버전: immutable 함수(코드 및 구성 포함): $latest버전을 기반으로 개발, 테스팅과 배포를 구분하여 게시, 버전을 활용해서 배포할 것
      - Alias: 버전을 가리키는 포인터(고유 ARN): API를 lambda alias에 매핑(매핑을 바꾸면 바로 대상이 바뀜, routing-config(람다 자체 기능)로 canary가능
      - SAM에서도 위 설정들에 배포패턴 적용가능, Alarm: 배포 성공여부 알람 설정 가능(최대 10개), Hook: 트래픽 전 후에 별도의 람다 적용가능(사전작업, 개발자 통보 등)
      - API말고 Event로도 가능?
  * CodeDeply + lambda
    + 배포 모델 지원, 클라우드 와치/알람 기반으로 롤백 가능
    + 디테일한 설정으로 배포 가능
  * API Gateway canary지원
    + API Gateway에서 버전 별 라우팅 비율을 조절, 이로인해 클라이언트의 변경은 불필요
    + Lambda자체의 canary와 API Gateway의 canary 차이
      - 단일함수 vs 전체 단계 수준
      - 서비스 호출 투명성 vs 클라이언트 호출 투명성
  * CodePipeline
    + Pull Request -> Lint/문법검사, 유닛 테스트, 컴파일 -> 테스트 환경에 배포 -> 스테이징 배포 -> 새버전 배포
  * 배포 모범 사례
    + 블루그린, 카나리를 활용
    + 다양한 버전을 필요로 할 경우 람다 버전을 활용
    + API 버전은 API Gateway버전 활용
  * 운영
    + 로컬에서 테스트 및 디버깅: SAM Local활용, 오프라인 동작, 라이브 코딩, 도커 기반으로 람다를 에뮬레이팅(깃헙의 aws-sam-local)
    + CodeBuild를 이용하여 람다 테스트: 도커를 이용하여 환경 구성
    + 전통적인 디버깅: 로깅으로 디버깅 -> CloudWatch -> 로그를 찾을 때 DynamoDB에 기록
      - X-Ray를 통해 리소스 간 호출 지도를 구축하여 모니터링 지원: 시각화 지원, 람다 설정 또는 코드 내에서 SDK호출
  * 정리
    + 자동 롤백, 배포 패턴 고려
    + AWS SAM + AWS CodeDeploy 활용하여 배포 모델링
    + 로깅과 모니터링 기능이 빌트인이므로 활용
    + X-Ray활용하세요(Lambda랑 잘 연결되어 있음)

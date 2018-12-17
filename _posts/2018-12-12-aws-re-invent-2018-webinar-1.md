---
layout: post
title:  "AWS Re:Invent 2018 요약 Webinar 정리 - 1부"
date:   2018-12-12 15:00:00 +0900
categories: AWS
comments: true
---
# AWS Re:Invent 2018 요약 (1부)
AWS Webinar를 들으며 요약했습니다. 급하게 적느라 빠지거나 틀린 부분이 있을 수도 있습니다.  
[AWS의 SlideShare](https://www.slideshare.net/awskorea)에 자료가 공유되어 있으니 자세한 내용은 직접 확인 가능합니다.  
AWS는 여전히 성장 중이며 굉장히 다양한 서비스를 제공
## 글로벌 인프라 업데이트
  * Region 확장 : 총 19개 region으로 확장 중 (5개 추가 중 - 홍콩, 케이프타운, 스웨덴, 밀라노, UAE)
  * Edge location도 많이 추가, AWS Global Accelerator 신규 서비스 - 바로 AWS 백본 망으로 연결하여 속도 증대
  * AWS Transit Gateway - VPC를 AWS 계정 및 On-premise와 손쉽게 연결(기존에는 VPC Peering, Direct connect 등을 통합)
## 컴퓨팅 업데이트
  * Instance
    + d가 붙어있는 인스턴스 타입 - 물리적 연결된 SSD로 속도 증대
    + AMD기반 인스턴스 추가 - 비용 10%정도 저렴
    + ARM 아키텍처 인스턴스(A type) - 비용 절감(45%?), 다양한 Workload 지원
    + n이 붙어있는 인스턴스 타입 - 네트워크 특화(고성능 대역폭 100Mbps?)
    + Elastic Fabric Adaptor - HPC(고성능 컴퓨팅) 네트워크 구축
  * Container
    + ECS, Fargate(서버리스 기반 컨테이너 관리), EKS, Fargate for EKS
    + Service Mesh - Side Car(Proxy) architecture, 서비스 옆에 라우팅, 모니터링, 보안을 해주는 기능을 덧붙임 (Istio)
    + App Mesh - ECS, EKS 기반 마이크로 서비스에 관리형 서비스로 서비스 매쉬 지원
    + AWS Cloud Map - 서비스 라우팅이 쉽게 되도록 지원
  * Lambda(Serverless)
    + IDE : Cloud9 + PyCharm, IntelliJ와 VSCode에 Toolkit for IDE
    + 지원 언어 : + Ruby(맞춤현 런타임을 통해 향후 그 외에 다양한 언어를 지원 가능)
    + 프로그래밍 모델 : 라이브러리를 함수 별로 패키징 해서 업로드 해야함, Layer를 통해 중복되는 코드, 라이브러리 등을 개별 함수에서 참조할 수 있도록 지원
      - 서버리스 앱 레포지토리(공유 및 판매) + SAM을 통해 기존앱을 Nested하여 재사용 가능 + ALB의 endpoint에 Lambda 적용가능
      - API Gateway에 WebSocket 지원 - 양방향 통신 응용 프로그램을 API Gateway에 적용 가능
    + 워크플로 : 기존 Step Function으로 함수를 순차적/병력적으로 수행 가능 - 여기에 다양한 AWS 서비스를 Step Function에 추가 가능
      - Kafka라는 오픈소스 MQ를 관리형 서비스로 제공(별도의 Zookeeper노드(관리용) 불필요), 3AZ에 배포 가능
  * Firecracker
    + AWS의 서버리스 및 컨테이너 구현은 VM방식
    + VM방식과 Container방식 2가지가 존재
      - Container 방식 - 보안이 어려움, VM방식 - 리소스 효율화 어려움
    + 기존 AWS Serverless는 VM방식 이었음
    + 이러한 방식(microVM)을 Apache 오픈소스로 공개 - Firecracker
      - KVM과 동일한 보안
      - 불필요 줄여 빠른 시작 시간을 위한 설계
      - 성능 최적화
## 스토리지 업데이트
  * Block storage : EBS
  * Object storage : S3 
    + S3 Intelligent Tiering(자동으로 접근 빈도 분석하여 class조절)
    + Glacier Deep Archive: Glacier 1/4저렴한 스토리지 추가
  * File system : EFS
    + EFS IA(Infreguent Access)타입을 지원하여 접근 낮으면 비용 절감
    + FSx for Windows: Windows 파일 서버 기반의 관리형 Windows 파일 시스템
      - Windows workload가 AWS는 57%, Azure는 30%, 기타로 점유
    + FSx for Lustre: Linux+Cluster, 고성능 병렬 분산 파일시스템 의 관리형 서비스
  * Data transfer : Direct connect, Snowball 
    + DataSync: 데이터 이동을 자동/가속화(10GMps)
    + Transfer for SFTP : 완전 관리형 SFTP기반 파일 전송 서비스

## 데이터 베이스 업데이트
  * Amazon Aurora : 최초 클라우드 네이티브 DB
    + Multi Master(작년에 출시) : AZ별로 여러개의 Master node
    + 글로벌 데이터 베이스 출시 : 여러 region에 걸쳐 클러스터 구축가능(각 region에 Master node, region 간 동기화)
  * Dynamo DB : 
    + Dynamo DB On-Demand : 트래픽에 따라 용량 자동 조절,과금
      - 기존에는 읽기 쓰기 처리량 예측이 어렵고, traffic spike 시 지연시간 발생
    + Dynamo DB Transaction : Transaction을 지원하는 최조의 NoSQL
  * 시계열(Time-series) 데이터 처리 : Amazon Timestream
  * Blockchain
    - Amazon Quantum Ledger Database : 관리형 원장 관리(투명한 암호화로 업데이트 추적가능)
    - Amazon Managed Blockchain : 블록체인 생성 및 관리(Hyperledger Fabric or Ethereum)

  

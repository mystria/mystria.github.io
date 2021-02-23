---
layout: post
title:  "AWS DevOps 교육과정 2일차"
date:   2018-11-10 09:00:00 +0900
categories: AWS DevOps
comments: true
---
# AWS DevOps 교육과정 2일차 정리
본 글을 2018년 11월 한국 AWS에서 교육받은 내용을 메모해 뒀던 글을 다시 정리하여 올린 글입니다.  
용어, 한글표기 등은 시간 관계상 맞추지 않았습니다.  

## 지속적인 통합(CI)
  - AWS에 관리형 서비스는 없음, 단 Jenkins같은 CI툴이 설치된 AMI가 있음
  - 코드관리 + 자동빌드/테스트
  - 코드관리 : AWS CodeCommit, 자체git, github같은 호스팅
* Code Commit
  - 설치 : AWS/IAM설정 > IAM에CodeCommit권한설정 > AWS CLI설치 > GIT설치 > GIT과 CodeCommit에 대한 IAM연결
  - EC2에 GIT 설치 후 SSH설정으로 비슷하게 구성 가능하지만.. HA제공은 어려움(S3, NFS, EFS(신규서비스))
  - 리포지토리 훅(hook) : 코드 리포지토리에 중요한 액션이 발생하면 이벤트 처리
  - Pre-commit hook : 커밋 전 코드 검증 등
  - Post-Receive hook : 푸시가 완료된 후 서버측 훅 - 자동 빌드 등
  - CI서버(Jenkins등)에서 버전변경 훅을 인지하여 최신코드를 pull하여 빌드 - 빌드fail시 개발자에게 알림 - 빌드 산출물은 공유
  - 안전한 CI서버를 위해 : IAM 제어, 방화벽 등
  - CI서버를 autoscaling : 필드가 많을 경우 자원이 부족 - 빌드작업이 많아지면 slave자원을 늘여 처리
  - 워크로드 분석 : 빌드개수, 빌드수행시간, 필요환경 - 나중에 instance 크래딧이용하여 일괄처리등도 가능
  - 서버없는 CI : Lambda, S3, DynamoDB, SQS, SNS

## 지속적인 전달(CD)
  - 전달과 배포까지
  - 자동화 / 설정값을 자원으로 표면화 / 커밋 빌드를 빠르게 / 실패를 빠르게 찾아 / 애플리케이션의 결합도를 떨어뜨림
  - 연관된 AWS 관리형 서비스
  - EC2, AMI, Route53, AutoScaling
  - CodeDeploy, IAM, CloudFormation, CodeCommint, ElasticBeanstalk, OpsWorks
  - 전달 시
    - 신규 서비스 런칭 시 시작 구성을 신규 서비스로 변경 -> ELB를 통해 auto scaling으로 신규 서비스로 이동(구형 서비스로 연결을 차례로 끊음) : 인프라 재사용은 비용은 절약할 수 있으나 롤백 어려움
    - ELB말고 Route53에서 가중치 기반으로 이동 시킬 수 있으나... 신규 자원 생성이 필요해서 비용 높음(기존 자원이 남아 있으므로 안정적 (블루-그린 배포방법)
    - A/B 테스팅 : 가중치 기반으로 여러가지 버전을 적게 올려두고 테스트
  - 배포전략
    - 실행중 변경 / 블루-그린 / 레드-블랙 / AB테스팅 / 다크(백그라운드에 신규 기능 배포)
  - 배포 기술
    - 무엇을 배포하나 : 코드 / 컨테이너 이미지 / AMI
    - Cloud Formation / Code Deploy(=애플리케이션 규모의 패키지를 EC2 instance의 agent가 배포)
  - 플랫폼
    - Elastic Beanstalk : 플랫폼은 구성되어 있고 애플리케이션만 설치
    - OpsWorks : 아키텍처 및 자원을 정의, Chef를 통해 코드/구성/패키지/DB/서버규모 등 관리를 자동화
  - 컨테이너
    - Docker Container : 규격화된 구성요소를 OS에서 알아서 배포
    - ECS : docker의 관리형 서비스 : 복잡한 클러스터 관리를 지원(ex:Docker가 auto scaling되었을때 클러스터 관리)
* Code Deploy
  - 애플리케이션 + AppSpec -> S3버킷 + GitHub -> Revision
  - Deployment Group에 Tag 기반으로 Revision을 받아와 배포(인스턴스에 설치된 Agent가 AppSpec을 참고하여 배포)
  - 배포구성 : AllAtOnce / OneAtATime / Custom
  - Lab의 작업 순서
    - 작업을 위한 role생성
    - 서버에 올릴 애플리케이션 파일과 appspec.yml, 그리고 deploy시 appspec에 의해 수행될 스크립트들(*.sh)
    - S3버킷 생성 : 여기에 리비전이 저장됨
    - code deploy용 application 생성
    - application에 속할 deployment group 생성(Console에서는 처음 d-g는 application생성시 동시에 생성, 나중에 추가되는 구조인듯)
    - 리비전을 만들 폴더로 이동 후 s3로 push
    - push하면 리비전이 생성되고 해당 리비전을 deployment하기 위한 명령어가 생성됨
    - deployment를 생성하면 config type(OnceAtAll 같은)에 맞춰 실제 배포가 이루어짐
    - get-deployment로 배포된 정보 획득가능
    - list-deployment-instance로 배포된 인스턴스 목록 획득가능
  - Lab의 Cloud Formation을 이용한 생성 작업순서
    - 스택 생성(탬플릿과 키 필요)
    - autoscalinggroup이름 획득
    - 위에서 생성한 application에 신규 deployment group 생성
    - 리비전 생성하여 s3로
    - deployment 생성
    - cloud formation의 autoscaling의 설정 변경
    - 반영된 내용 확인


## Elastic Beanstalk
  - 간단(용량/로드밸런스/규모.모니터링 등을 제공) 하지만 설정할 수 있는 구석이 별로 없음
  - CodeDeploy vs ElasticBeanstalk
  - EB가 stateless환경에 더 적합
  - 큰 환경 설정이 필요할때는 CodeDeploy + CloudFormation이 적합

## AutoScaling
  - CloudWatch / 일정 / 사용자지정
  - 시작 구성(Launching configuration) > Auto Scaling Group > Policy
  - Route53
  - DNS 장애극복
  - 가중치 기반 라우팅 : 시스템 업그레이드 컷오버에 유용
  - 지연 기반 라우팅 : 사용자와 가까운 지역으로 전달

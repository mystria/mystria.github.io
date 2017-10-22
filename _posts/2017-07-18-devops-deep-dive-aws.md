---
layout: post
title:  "AWS DevOps Deep Dive 세미나"
date:   2017-07-18 14:00:00 +0900
categories: AWS DevOps
comments: true
---

# AWS를 이용한 DevOps Deep Dive 세미나
1. [개요](#개요)
2. [DevOps](#devops)
3. [Infrastructure 관리](#infrastructure-관리)
4. [Summary](#summary)

## 개요
본 포스팅은 2018년 7월 18일에 있었던 AWS 301(이 숫자가 뭔지는 아직 모름...) 세미나 에서 진행한 AWS DevOps Deep Dive 감상(?)이다.
혹시 저작권 문제가 있을 수 있으므로 세미나 요약이 아님을 밝힙니다.
  * 총평 : 아무래도 AWS DevOps에서 "DevOps"보다 "AWS"가 주가되어 AWS의 제품들을 홍보하는 느낌의 세미나. 그러나 Immutable Infra의 개념과 AWS에서 점점 강화되고 있는 CI/CD 도구들을 알게 된 것으로도 충분히 만족스러웠음.

## DevOps
  * 개발 문화
    + 기존의 중앙에서 Approval 하는 식의 개발은 지양
      - 예전에는 릴리즈를 주기적으로했으나 요즘은 가능한 많이 함
      - 예전에는 고객에게 영향을 최소화하고자 했으나 요즘은 아무때나
      - 예전에는 문제가 있으면 릴리즈 이전으로 롤백했으나 요즘은 재수정 후 덮어씀(포워딩)
    + 지속적 통합(CI) -> 지속적 전달(CD) -> 지속적 배포(=지속적 전달)
      - CI/CD에서 휴먼에러가 가장 큰 문제, 잘못 설계하면 지속적 에러가 발생 - 검증된 도구&sup1;를 쓰게 된다.
      - 자주 + 작게 배포를 하면 에러 크기가 줄어들어 스무스하게 발전할 수 있다.
    + DevOps란? 소프트웨어 개발 주기를 단축
    + Amazon에서는 어떻게 개발할까?
      - CI/CD는 필수(자주 커밋, 모든 커밋을 빌드, 테스트위해 실행환경 배포), 모든 코드화 가능한 것은 저장소에 저장, 지속적 전달
      - 카나리아 배포, AZ배포, Region배포
      - 코드리뷰 필수, 코드 스타일(Lint) 검사
      - 자동 롤백을 중요하게 여김
      - 대시보드를 철저하게 관리
      - Code 시리즈는 IAM, Lambda, CloudFormation과 잘 연동되니 잘 활용
  * AWS 서비스들(Code 시리즈와 CloudFormation)
    + CodeCommit - 코드 저장(관리형 Git, S3기반(비용/안정성 높음))
    + CodePipeline - CodeCommit또는 Git으로 부터 코드를 가져와 빌드하고 (EB&sup2;같은데)릴리즈, 신속한 업데이트
      - stage = action + action(병렬), stage간은 transaction이라고 부름
      - 수동 승인 기능(SNS을 통해 알림)을 추가(팀 내에서 승인해야 진행됨)
      - 배포시 CloudFormation(+Parameter)을 선택할 수 있음
      - CloudWatch를 1분단위로 하여 체크를 하고 이를 SNS를 통해 Pipeline을 멈출 수 있다.
    + CodeBuild - Jenkins와 비슷한 관리형 서비스, 빌드시간 분단위 과금
      - Docker 이미지를 통해 빌드 환경이 로드되고 빌드 스크립트 수행
      - 기본 제공하는 Java, Python 등도 있지만 준비된 Docker 이미지가 있으면 다른 환경도 적용 가능(docker 로드가 엄청 빨라서 괜찮아~)
      - 빌드는 buildspec.yml(install, lint, test, build, artifact 같은 단계)로 정의
      - Jenkins와 통합 가능 : Jenkins를 master로 쓰고 CodeBuild를 worker로 하여 클러스터화 해서 사용가능(보통 Jenkins로만 클러스터로 쓰지만, 이것보다 비용효율이 좋을 수 있음)
    + CodeDeploy - 자동배포, Rolling배포/Blue-Green배포 지원, ELB 또는 AutoScalingGroup 활용
      - 인스턴스를 등록하고(AutoScalingGroup, EC2뿐 아니라 온프레미스 서버에도 agent설치하면 가능) 어떤 정책으로 인스턴스 들에게 디플로이 할 것인지 정의 가능, tag로 설치할 대상 인스턴스를 식별함
      - 배포는 appspec.yml로 정의 : 소스 저장소에 위치할 수 있고, 단계별 후크 스크립트 지정(배포 성공 여부를 후크를 통해 유효성 검사)
    + CloudFormation - 인프라를 템플릿으로, 코드처럼 관리, 의존성에 따라 프로비저닝 할 수 있음
      - 최신업데이트: yml지원(예전에는 json), 변경내용을 미리 볼 수 있는 change set, Export를 통해 스택간 상호참조(Stack을 분리하여 관리할 때 스택간 참조, Nested Stack 스택안에서 다른 스택 로드)
      - 스택들을 위한 지속적 전달 워크플로우(CodePipeline에서 지원), SAM(Serverless App Model)지원

## Infrastructure 관리
  * Immutable Infrastructure : 불변하는 인프라
    + 개발하다보면 환경이 금방 엉망(?)이 된다. 이때는 인프라를 변경하지말고 새로 만들어라. -> 배포할땐 새 인프라에!
    + 보통 CLI나 SDK를 통해 스크립트로 AWS 인프라를 관리 : 롤백, 시간측정 등이 어려움
    + 인프라와 소프트웨어를 함께 관리, 언제든지 재현할 수 있어야 하고, 보안 등을 통해 감사가 가능해야 함 -> CloudFormation
  * CloudFormation의 장점 : 인프라를 코드로 관리하여 특정상황(최신 버전의 인프라와 다를 때)에서 옛날 버전을 로드하여 사용가능
    + CloudFormation에서 application 배포 시 - 부트스트래핑 활용
  * 피드백루프 : CloudWatch
    + CloudWatch logs 사용(중앙 집중식 로깅) : 비용 발생 주의(Full logs 조심), 필터 활용
    + X-Ray : 코드 내(API 호출인듯)에 위치하며 정보수집 가능

## Summary
  * 전체적으로 보면 CodePipeline이 조율하고, CodeCommit에서 소스관리, CodeBuild에서 빌드, CodeDeploy에서 배포
  * 인프라 구성은 CloudFormation을 무조건 쓰고, 간단한건 EB로 해결
    + OpsWorks는 서버내 러닝 환경을 쉐프를 통해 스크립트화
  * 혁신은 민첩성이 중요
    + 배치 축소 및 지속적 전달을 위한 노력
    + 혁신 = (문화 * 구조) ^ 도구화
  * 즉, AWS의 도구들을 쓰세요!

---
&sup1; AWS의 Code 시리즈를 쓰라는 암시(?)인듯  
&sup2; EB : Elastic Beanstalk, 애플리케이션 레이어 중심의 PaaS서비스

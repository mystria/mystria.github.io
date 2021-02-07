---
layout: post
title:  "AWS DevOps 교육과정 3일차"
date:   2018-11-11 09:00:00 +0900
categories: AWS DevOps
comments: true
---
# AWS DevOps 교육과정 3일차 정리
본 글을 2018년 11월 한국 AWS에서 교육받은 내용을 메모해 뒀던 글을 다시 정리하여 올린 글입니다.  
용어, 한글표기 등은 시간 관계상 맞추지 않았습니다.  

## OpsWorks
- 앞서 말한 EB같은 것은 1레이어 생성이 가능하다면, 보다 통합적으로 관리 할 수 있음
- 세밀한 컨트롤 가능(Chef)
* 개념
  + Stack : cloud formation과 동일 - 관리할 자원의 집합
  + Layer : 인스턴스, 자원의 구성 - OpsWorks(EC2 인스턴스)계층과 서비스(AWS관리형 서비스) 계층
  + Scaling : 
  + Chef recipes : OpsWorks로 생성된 인스턴스에는 chef용 agent가 기본설치됨
* 
  - OpsWorks서비스가 인스턴스의 생명주기를 관리하고 상태를 확인할 수 있음.
  - 인스턴스에 agent가 있어서 레시피를 수행하고 상태를 OpsWorks로 보고함
  - 인스턴스가 5분이상 연결이 안되면 인스턴스 재시작 가능 - auto scaling과는 살짝다름 (AS는 인스턴스 삭제후 재등록)
  - Chef Cookbook - ruby 언어
  - Attribute file + Template + Recipe
  - http://docs.aws.amazon.com/opsworks/latest/userguide/cookbooks-101.html
  - OpsWorks + DevOps 통합
  - OpsWorks는 완전 스크립트 화 가능
  - 실행중 업데이트 가능 - cookbooks 전달(어제 강의처럼 롤백의 어려움)
  - 블루-그린 배포 가능
  - OpsWorks + Cloud Formation
  - CF로 인프라를 구성하고, OpsWorks로 애플리케이션 계층 배포

* 비교
  + Elastic Beanstalk : 완성형 서비스
  + OpsWorks : EB와 CD의 중간
  + CodeDeploy는 인프라 구성 불가 - CF랑 같이 써야됨

## 컨테이너
  - 컨테이너는 하나의 프로세스라고 생각하면 되지만, 각 프로세스간의 격리가 이루어짐
  - 가상화를 통해 호스트OS레벨에서 격리 - 커널은 공유
  - 다양한 앱이 각자의 파일시스템, 저장소, 램을 가짐
  - 장점
    - 개발및 배포가 단순화, 패키징 해두면 어디서나 실행가능
    - EC2 사용률 극대화
    - ELB가 ALB(Application Load Balancer)가 되어 분배해줌(port ?)
  - Docker는 외부기업(Docker 허브 : 이미지를 등록해 둘 수 있음(유/무료))
  - 프라이빗 레지스트리 (CF로 템플릿 제공)
  - AWS ECR(EC2 Container Registry)
  - 이미지(이게 컨테이너들의 구성인듯) = Docker file = Task
  - 규모가 커지면서 관리 문제가 발생
  - 복수 컨테이너 / 복수 버전 / 복수 인스턴스 -> ECS
  - ECS
    - Task : (=Docker file) 
    - Task definition : 작업 정의
    - Run task : 1회성 작업 / Service task : 지속적으로 돌아감

* 배포방법
  - Zip, repository로 빌드, AMI, Docker컨턔이너
  - AMI : Application AMI(최신 운영버전의 앱 설치된) > Base AMI(목적에 맞게 최적화) > Foundation AMI(기본적인 설정) > AWS AMI(AWS에서 제공)
  - AMI : Custom > Gold > Silver

## Code Pipeline
  - 빌드를 해주는건 아니고, CI CD를 조율
  - 소스코드(Git..), Build(Jenkins...), Test, Deploy(EB, CodeDeploy,,,), Invoke(커스텀 함수를 Lambda로 제공)
  - Lab에서 보면
    - S3에 git소스가 push 되면 code pipeline에 트리거
    - CP에서 해당 소스를 인지하여 codeDeploy로 배포
    - 다음단계로 테스트 수행(using Jenkins의 execute shell)
    - Jenkins가 feedback을 주는가?(Jenkins의 Post-Build인거 같은데…)
    - 성공되었다면 다음 단계 CP 수행
    - CD 실행
  - 위와 같은 방식으로 반복되어 흘러감

## 튜닝
  - 성능 튜닝
  - 남의 튜닝 충고를 그대로 따라하는 건 남이 먹는 약을 그대로 먹는것과 동일 -> 직접 테스트
  - 운영과 동일한 환경 구축, Scale in/out 점검
  - 테스팅 도구 사용
  - ELB튜닝
  - 크로스 AZ
  - DNS 캐쉬하지 않음
  - 애플리케이션 핼스체크의 빠른 응답
  - Route53의 alias사용
  - ELB단 이후로 SSL 종료(안쪽은 신뢰있다고 가정)
  - Auto Scaling
  - 지표는 1개만 가능(CPU, Mem), 만약 두개를 동시에 보려면 커스텀agent를 설치하여 알람으로 noti
  - T2는 크래딧이 쌓이므로 이를 지표로(크래딧이 없으면 성능 10%만 사용가능하므로 크래딧을 체크해야함. 크래딧이 남도록 운영해야 함 - 큰 사이즈)
  - 쓰레싱(스케일 인아웃 설정 미스로 계속 확장 축소 반복) 감시
  - sticky session지양
  - AutoScaling 이슈 - 빠르게 확장되지 않을 경우
  - 상세모니터링 활성화
  - 샘플링 기간
  - 쿨다운 기간
  - 규모 조정 단위(늘어나는 요청 만큼 크게 키우는지)
  - 갯수 단위로 확장 / 비율단위로 확장
  - AutoScaling 이슈 - 규모 조정이 안됨, 쓰레싱
  - 고급 관리
  - 확장/축소 이벤트 시 어떻게 애플리케이션을 배포할지 고려
  - 생명주기에 맞춰 훅 사용
  - EC2 적합한 성능 찾기 - 카나리아 방법 : 인스턴스 한개를 ELB에서 바꿔봄. 해당 인스턴스가 적합하면 점차 다 바꿈

  - 클라우드 와치를 쓰려면 EC2에 agent가 설치되어 log를 발생시켜야 함

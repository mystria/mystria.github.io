---
layout: post
title:  "Korea DevOps Meetup 2018"
date:   2018-01-13 12:30:00 +0900
categories: DevOps
comments: false
---
# KeyNote - 공준
What is DevOps? 일을 효율적으로 하기위해 반복적으로 하는 행위들
  * Collaboration: 큰 의미의 협업, Affinity: 친밀도, Tools: 도구, Scaling: 점점 커져감
  * 결국엔 Tools: 도움이 되기도, 방해가 되기도...
    + Local Development Env, Version Control, Artifact Management, Automated Test
      - 카카오는 Kfield, Jenkins Lake, Kitchen, Chef ...
  * 자동화: 왜? 자동차의 부품들이 처음엔 수동이었지만 자동화 되고 있음, 나아가 자율주행
    + Sense: Resource measuring
    + Plan: Threshold scheduler
    + Act: Provisioner
    + 이런 것들을 통해 Data Center의 자원들을 자동화 하고 향후 AI까지..
      - 그리고 분석 -> 피드백 반복해서 발전

## 10 Ways Kubernetes Enables DevOps
김현수(RedHat, hykim@redhat.com)
DevOps: 시장에 빠르게 제품을 출시하기 위해, Agile로 부터 나온 개념
Collaboration culture, Delivery automation, Programmable Platform
운영과 개발간의 충돌을 완화하기 위해선 문화도 중요하지만, 도구와 플랫폼의 지원이 필요 - 안정적 운영으로 상호 협업이 잘 될 것이다.
Kubernetes는 거대한 컨테이너 앱 관리용 오픈소스 프레임워크(디팩토스탠다드)
레드햇은 Openshift제공
참고: 10가지 기능들을 알려주지만 이게 전부도 안고 필수도 아니다
1. 배포 자동화: 어떤 인프라에서도 컨테이너의 자동 배포
2. Infrastructure as code: 선언적(declarative) 컨테이너 인프라, 장비를 수정하지 말고 코드를 수정해서 관리
3. Configuration as code: 선언적 앱 설정을 ConfigMaps and Secrets를 통해 설정들 역시 코드로 관리하고 kubernetes로 적용
4. Immutable Infra.: 인프라가 변경되면 앱이 깨질 수 있고, 변경을 놓칠 수 있다. 컨테이너 이미지는 변경하지 말고 새로 만들것 (또는 layer를 하나 더 올려 관리하든가..)
-> 이는 서버(서비스, 컨테이너)가 동일한 상태를 유지하려는 습성, 즉, Reliable
5. On-Demand Infra.: Self-Service Catalog로 부터 온디맨드로 인프라를 생성
6. Environment Consistency: 한번 만들고, 여러 환경에 동일한 모양으로 배포 +  설정파일로 가변
7. Continuous Delivery Pipeline: 파이프라인을 만들고 젠킨스로 스케일, Kubernetes는 Jenkins뒤에서 Deploy, Verifym Release, Golive까지 담당 : 이거 우리가 하는 일이네?
8. Zero-downtime Deployments: 운영 트래픽에 방해없는 안전한 롤링 업데이트 그리고 롤백, (피닉스 배포?). 운영에 청록 배포, 카나리 배포 기능 제공
9. A/B Testing: 카나리 배포 비슷.. 운영에 용기를 주자? Encourage to Business Outcomes
10. Cross-functional Collaboration: Share access? 못들었네..
이런 10가지 일이 일어날 수 있도록 지원해주는 기능
말로는 DevOps를 정착 시킬 수 없다. 도구가 필요
DevOps adoption needs the right technology mix, Kubernetes pave (??)
공공기관 같은 특별한 케이스: 개발과 운영이 다른 조직(부처와 센터)으로 물리적 분리

## 그들이 AWS 위에서 Kubernetes를 운영하는 방법
ZEPL 1ambda.github.io
왜 쿠버를 쓰는지? 스타트업에서 인프라 엔지니어가 없다, 적은 지원/자본, 대안이 없다
새기능 넣기도 바쁘다, ansible로 관리하기엔 복잡.. 최대한 쉽고 간단하게 할 수 있게 하자
타임존이 다른 개발자, 롤백을 어떻게 하는지 알 수 없다.
고객이 다양한 인프라를 쓰고 있다, 컨테이너로 가자 - 오케스트레이션 레이어
컨테이너 레이어의 오디팅, 과금 : 쿠버네티스에 이미 지원
KOPS(Kubernetes Operations)
AWS EKS가 준비중이지만, 아직 여러 리스크 존재, KOPS를 쓰자

HA지원, Terraform파일을 생성해줌, CNI선택가능, AutoScaling, Spot instance지원
직접 관리하면, 업그레이드, 시큐리티, 사용자, 옵션 등을 챙길게 많음

쿠버네테스는 클러스터 덩어리- 노드와에 워커와 마스터.. EC2 인스턴스 같은거..
클러스터를 vpc-subnet-az에 마스터와 워커를 배포하고, 클라이언트가 접속할 API 서버와 도메인 필요, IAM 준비, VPC에 dns 활성화, Subnet설정 multi-az를 위해 tag 관리 필요
클러스터 산출물을 테라폼으로 생성하여 plan - apply
Federation과 중복 컴포넌트 준비 HA
큰 인스턴스를 쓰면 많은 Pods을 띄울 수 있음

워커 노드에 팟을 띄우고 그 위에 컨테이너를 띄움
클러스터 내부 IP는 외부와 접근이 안되므로 AWS 내부 DNS를 써야함
컨테이너에서 k8s의 IAM권한을 사용할 수 있으니 조심해서 설정할것
k8s의 ingress라는 것을 이용해서 ELB하나만 외부로 내보내고 내부통신을 할 수 있음, 찾아볼것
nginx ingress controller 추천, traeker?는 잘 안됨

모티베이트
- 아무나 배토할 수 있게
- 각자 배포하고 테스트
- 배포 알아서 하게 노티스
- 본인이 쉽게 롤백
git-젠킨스-도커-슬렉-k8s

팟 숫자를 조절하거나, 팟 내부 설정들을 할 수 있음
k8s를 여러 사용자들이 사용할 수 있게 사용자 인증 설정, x509 cert가 쉬움
dashboard를 설치하기 위해서는 heapster를 설치해야함, 인증이나 보안 설징이 복잡하므로 readonly로 설정하고, 작업은 kubectl로만 허용
로그는 daemon set을 이용해서 agent를 배포, 별도의 프로세싱이 필요할 경우 sidecar를 구축하여 처리
k8s 시스템 로그는 범용 플러긴으로 확인, add-on을 지원함
kubectl

교훈들
HA는 돈이 너무 많이 든다
보안이 어렵다 - auditing, etcd, IAM for KOPS
팀, 도구, 문화 - 굳이 새 도구를 써야하는가? 좋은 것이라고 강요할 수 없음. 도구가 아니라 문제에 집중할것, 사람들이 이 도구와 문화에 적응하도록 노력

goo.gl/F23R79

## 이슈 해결로 알아보는 DevOps
Content Delivery 중점으로 설명
개발팀은 이상적으로 생성
운영팀은 패널티 기반으로 성과
이를 조화시킬 KPI 필요
현실은 달리는 자동차 바퀴갈기
DevOps - Lean Startup
  - 돌아가는 작은 것부터 만들고, 키우자. 일단 움직이고 리스크는 있을 수 밖에 없음
개발자를 뽑아 운영을 하면 어떨까?
규모가 커지고 복잡해지면 관리 시스템이 필요해짐
Consistent Hash 도입 사례 : 작은 이미지(롱테일)가 너무 많다, 용량이 큰 동영상도 늘어간다
몸빵으로 버티는것도 힘들어진다.. 인프라에서 할 수 있는건 다 해봤으나.. 더 나은 방법은 없나?
-> Consistent hash 도입, 이거 선형 검색을 원형으로 바꾼거네.. 이거 서버 캐쉬에도 적용할 수 있겠네 - 즉, 하다보면 운 좋게 얻어걸리는게 있음 - 찾아보면 선지자들이 있음
인기 동영상 인덱싱 사례 : 동영상에 카운팅을 하여 구분하여 보관, 근데 이걸 이미지에도 하려니 안됨(요청향이 많고 레이턴시에 민감)
-> 현실과 이상의 밸런스, 번아웃 조심,
또 다른 사례 -> 프로세스를 바꾸는건 힘들었다. 자동화도 좋지만 너무 크게(비대하게)갈 필요는 없는거 같다
결국에는 클라우드 마이크로 서비스처럼 되더라... 사람들의 고민과 해결은 비슷하지 않을까?
90%정도는 맞춰가지만 10%정도는 내려놓고 그때 그때 해결해도 좋겠더라
장시간 집중해야하는 프로젝트에는 데브옵스가 불필요
루틴한 일은 운영팀에 이관
마무리
운영/개발/기획을 굳이 날카롭게 나누지 말고, 오픈마인드로 넓게 바라보자
러닝 커브로, 처음엔 느리다. 이걸 기다려주는 시간이 필요

## 테라폼 Terraform 도입기 - 클라우드 인프라스트럭쳐 코드로 재정의 하기
김대권 @nacyo_t SMARTSTUDY SRE팀
* 코드로서의 인프라스트럭쳐
  Infrastructure as Code, IaaC
  + 물리적환경, 가상자원 등 리소스의 집합
* 코드
  + 서버 스크립팅(오늘 이야기X)
  + 설정 관리
    - Chef, Puppet, Ansible : DSL, 멱등성 지원
    - 인프라가 코드가 되면 Git과 같은 곳에서 관리, SW개발과 마찬가지로 프로세스로 관리됨
  + 리소스 선언 도구
    - Terraform, CloudFormation : 클라우드 가상화 리소스를 정의

* 테라폼 Terraform 소개와 HCL기초
  + Terraform : 인프라 관리 도구, 의존성 표현으로 리소스간의 의존적인 생성 가능
  + HCL: HashiCorp Configuration Language, HashiCorp 서비스에 사용할 수 있는 코드
  + 워크플로우 : Write - Plan - Create(apply)
    - 정의.tf -> 실제배치.tfstate
  + 테라폼 사용방법
    - 새로 만드는 리소스를 테라폼으로 작성
    - 기존 리소스의 임포트 : tfstate로만 가져옴, 근데 tf가 없으므로 동기화 시키면 날아감
    - terraforming 라이브러리 사용 : tf, tfstate로 임포트, 완벽하지 않고 모든 리소스를 가져오지 못함, 나중에 수작업 보정 필요(의존관계 등)
  + 테라폼 도입 이유
    - 코드화로 변경사항 추적관리 가능
    - 인프라 롤백 및 변경의 부담 감소
    - 고도화 작업 : IAM관리, 태그관리
    - IAM관리 사례 : 퇴사자, 정체불명의 롤, 모두 슈퍼어드민 - 테라폼으로 IAM리소스도 관리, 미사용 롤, 권한 삭제, 점진적으로 사용자 권한 축소, 웹콘솔 사용 빈도 줄임
    - 태그관리 사례 : 태그는 유용하고 좋지만 자동화가 아니면 태그를 남기기 힘듦, 필수/선택 태그 구분 후 필수는 반드시 남김, 태그는 리소스별로 문자규칙이 다름, 태그없는 리소스도 존재

* AWS 리소스를 Terraform으로 재정의
  + 임포트 하자
    - 모든 리소스를 하나로 관리하고자 하는 유혹 : 너무 느려짐,
    - 다수의 테라폼 프로젝트로 분리하여 구성
    - common한 자원은 분리하여 정리, 다중 vpc
    - 프로젝트 간 리소스 공유를 output이란 것으로 간접적으로 가능(직접적으로 안됨: 거대한 프로젝트의 장점)
  + 협업
    - 리모트 스테이트란 걸로 tfstate를 공유, S3등으로 관리, 동시작업을 막아야함(dynamoDB를 활용하여 락킹)
    - 이후 소프트웨어 개발 방법 활용(Github 등)
    - 코드로 수정, 이슈로 제안, Pull Request시 SRE팀 리뷰, Merge
  + 자동화
    - CI : fmt로 코드 스타일 강제, plan결과 알려줌, apply는 장담할 수 없어 적용하지 않았음(실패시 대응 프로세스 필요)
  + 단점
    - 별로 없고 초기도입의 어려움
    - 리소스 레퍼런스 익히기 어려움
    - 자동화가 잘 안됨
  + 장점
    - 코드 기반 협업
    - 변경 기록 -> 노하우가 쌓임
  + 미래
    - 컨테이너 기반
    - 불변 인프라, NoOps
* 추상화가 가능할까
  + 클라우드 기반으로 모든 리소스의 추상화
  + 테라폼 모듈 레지스트리

terraform
  common
  service1
  service2
참고: r.nacyot.com

## Docker와 Azure Container Service를 활용한 DevOps 구현 사례
On-Demand상태의 Microservice를 Container 환경으로 Migration
1. k8s + registry
2. 로컬 개발환경
3. DevOps : Microsoft Visual Studio Teams Service
데모 환경 : Github + Dokcer + Java Spring boot
CI : git 소스 가져오기 - 빌드 - k8s config
CD :

Azure Container Service : K8s, DC/OS, Swarm
MS Azure에 k8s를 디플로이하는 데모 : 5분만에 구축됨, k8s를 Azure에 맞게 구축하려면 엄청 힘들지만, AKS를 활용하면 편함
Azure는 리소스 그룹으로 한번에 생성된 것들을 표시해주네..
레지스트리도 제공해줌
Managed K8s on Azure도 서비스 시작: VM 가격만 내면 됨, Master Node는 관리형으로 제공함

로컬에서 메이븐으로 빌드하고 도커이미지를 만들고 도커컴포져로 띄우는 데모

이 과정을 VS teams services로 CI/CD할 것
Jenkins의 MS버전, 플러긴 많음
빌드 후 k8s에 배포 - 간단한 배포(kubectl apply, 간단한 설정)

Azure 사용하지 않아도 teams 쓸 수 있나?
비용은? 무료버전을 기업에서도 쓸 수 있나? Azure상관없이 5인 무료 MS계정

## DevOps를 위한 모니터링
IT Monitoring
범위 : 인프라, 애플리케이션, 네트워크(클라우드환경이 되면서 좀 약해짐), 사용자
사용자 : IT운영자, SysOps, 네트워크관리자
개발자와 운영자가 점점 구분이 모호해지고 있음
현대IT목표 : 더빠르고 안정적으로
고전적인 모니터링이슈 변화 - 애플리케이션 모니터링은 초반에 중요도가 높지만 갈수록 미미해짐(대충 아는 이슈만 발생), 이로 인해 시스템 모니터링은 안정화 단계로 갈 수록 더 중요해짐 - 이래서 개발자들은 모니터링에 무관심해짐
최근 모니터링 이슈 - 일단위 배포같이 주기가 짧아지면서 애플리케이션 모니터링이 중요해짐(언제든지 문제가 발생할 수 있다) - 개발자와 운영자가 함께 서비스 분석을 하게됨: 개발자가 운영에 참여하게 됨, 클라우드로 전환되면서 개발자도 시스템 모니터링이 가능해짐 - 사실상 구분이 불필요해짐
모니터링이 개발팀과 운영팀의 다리가 됨

APM(Application Performance Monitoring)분야 대두 - New Relic, Datadog, 주요기업들이 퍼포먼스 모니터링을 채택중
APM이란?
사용자의 트랜잭션을 추적 : 사용자가 요청하고 응답받은 시간속에서 왜 늦어졌는지 내부 흐름을 추적 - 튜닝을 할 위치를 알려줌
New Relic : 추세선, 개발자가 text로 보는 걸로는 부족
HitMap형식의 모니터링 : 사용자별 응답시간을 점으로 표시함
개별 응답의 내부를 추적하여 어디서 어떻게 속도가 늦어지는지 확인가능. 개발자도 자기 코드가 어떻게 호출되고 딜레이되는지 알 수 있음, 튜닝하려면 이런 추적이 필요, 개발자가 여러명이거나 바뀌었다면 필수
호출되는 메소드들을 누적후 분석
서비스 배치 후 성능 변화 체크 가능(New Relic)

DevOps가 APM을 사용하는 방법
1. 개발자-운영자 동일한 APM사용
2. 운영이 코드 수준까지 진단, 문제개선
3. 개발과 운영이 배포 전후 성능 변화 분석
4. 장애 상황에서 자동으로 실행 가능한 설정을 이용 - 메모리 정리, 오토스케일 등을 자동화하라, 그리고 성능 개선 후 재배포

APM 사용을 통한 DevOps사례
1. 기존 개발환경에서 1~2주 테스트 후 업데이트
2. APM도입 후 일간 서비스 업데이트
3. 일간/주간 성능 비교통해 서비스 성능 체크 - 이걸 하지 않으면 언제 왜 느려졌는지 알 수 없음
4. 장애나 성능 저하시 운영/개발에 노티
5. 개발팀은 기능 변화에 따른 인프라 증설을 APM으로 측정하여 운영에 미리 고지

시스템 - 데이터베이스 - 애플리케이션 - 인테그레이션 포인트 - 퍼포먼스 - 유저 행동 - 비즈니스 프로세스
각각 모니터링에서 통합 모니터링으로 시장 변화
모니터링 서비스에서는 애플리케이션과 인프라를 통합해서 모니터링하는 것으로 변화
NoOps: 별도의 운영말고 개발자가 모니터링
모니터링없는 DevOps은 헤드라이트 없이 밤에 운전하는 것과 같다.
APM을 헤드라이트로 사용해라

APM을 위해 개발단계에서 뭔가 라이브러리나 agent를 추가해야되는지? WAS서버에 agent설치필요
모니터링 업체에서는 메트릭을 대부분 강제(업체가 중요하다는 것을 suggestion을 하는 방식)

DevOps는 이제는 필수, Agile과는 다르게 문화보다는 도구이므로 거부감이 적다. Startup에서는 운영자가 없어서 존재, 대기업들도 DevOps 조직을 만드는 추세

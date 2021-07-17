---
layout: post
title:  "Google Cloud Summit 2018 Seoul"
date:   2018-10-25 09:00:00 +0900
categories: GoogleCloud Summit
comments: true
---
# Google Cloud Summit 2018 Seoul
Google Cloud Summit 2018 에서 참석한 세션입니다.

## 기조연설
2018년 10월 25일 COEX

* IoT
  - TPU를 통한 엣지 컴퓨팅 파워
  - 서버리스를 통한 머신 러닝, 인텔리전스
    - 예방 정비 같은 분야에 활용 - 센서정보 수집, 서버리스 분석, 시각화
  - 하드웨어로 부터 시작하는 보안 - 자체 제작
  - 인텔리전스 IoT로 넓은 산업 분야에 적용
  - 스마트 시티 : LG 전자 - 인공지능 가전 > 스마트 리빙 > 지역 업체와 공유(O2O), 에너지 관리(스마트그리드), 스타트업 창업/육성 지원 - 국제 업무 단지

* ML(인공지능)
  - AI를 적용하여 고객경험 개선 - 구글 모든 제품에 탑재
  - 텐서플로우 기반 - TPU(Tensor Processing Units) 개발, 기존 CPU 보다 빠름
  - 누구나 쓸 수 있게 자동화 제공 - 사전 제공되는 모델 활용 - 데이터만 넣으면 됨 > 자기 모델 생성도 가능
  - AutoML로 복잡한 과정 생략 - 데이터 수집 후 예측(중간과정 없음)
  - BigQuery를 통해 SQL쿼리를 이용해 초 대용량 데이터를 간편하게 탐색

* 인프라
  - 서버비용 감소, 관리비용 증가 > 비즈니스에 집중할 수 있게
  + 컨테이너 배포 > Kubernetes > GKE(관리형 Kube)
  + GKE On-Prem을 제공(사용자 데이터 센터 + G 관리)
  + Istio : 컨테이너 상의 안전하고 일관된 데이터 통신, 트래픽 관리, 모니터링
  - 네트워크 단위 서비스 - 더 우수한 네트워크 환경
  - 클라우드 아머 - DDoS같은 공격 방어

* 업무 방식
  - 기업 임직원의 단순 작업에 따른 손실(자료 관리, 공유 등) > 새로운 업무 방식 필요 > G Suite
  - 보안 + 스마트 + 단순 제공

* 개발자
  - VM(GCE) - Kubernetes(GKE) - PaaS - Serverless(GCF)
  - Apigee - API제공 > 개발자 에코시스템 제공
  - Knative - 오픈소스 - 커뮤니티 - Kubernetes로 서버리스 애드온(GKE)
  - 쿠버네티스를 통해 뭔가 개발자 생태계를 제공하고자 함

* 변화가 새로운 표준 - 양자 컴퓨터(퀀텀)으로 가는 중


## 클라우드의 CICD
  - CI는말고 CD위주로..
  - 지속적인 딜리버리 : 변경을 자주, 안정적으로, 프로덕션에 전달
    - 변하는 것들
    - 변화의 가시성
    - 변경 실패관리

  * Spinnaker
    - 오픈소스, 모든 주요 클라우드 지원(각각 프로바이더가 직접 참여)
    - 사실상 엔터프라이즈 CD 솔루션

    - 삼성전자 무선사업부 Spinnaker 담당자(DevOps, SRE)
    - 끊김없이 사용할 수 있도록 고민…247 사용자가 눈치 못채게
    - Chef로 인프라 관리 - 피닉스 서버
    - 스노우플레이크(사용자가 일일이 작업하면 눈꽃처럼 모든 서버가 달라진다)
    - immutable 서버로 안정적인 서버 관리
    - 위를 도입 하더라도 적은 인원으로 많은 서버 관리는 어려움
    - Spinnaker의 도입
      - 4개의 스핀에이커로 2600개 이상의 서버를 관리 중 - 100개 이상의 마이크로 서비스
      - 배포도 중요하지만 실패 회수도 중요 - 스핀에이커는 여기에 강점
      - 블루그린 지원(ASG와 LB사용)
    - Spinnaker의 장단점
      - 사내 데이터 센터, 모노리틱, 리소스의 히스토리 관리 부재
      - 자동화 도입 - Ansible, Packer, Terraform, Spinnaker
      - 인프라 구축, 테스트를 거친 이미지 배포
      - Stage가 production과 동일한 환경이어서 검증이 원할
    - Spinnaker를 통해 파이프라인 구축
    - fency한 UI
      - 정교하고 다양한 기능 지원 (마치 방망이 깎는 노인처럼 관리/보강 가능)
      - 오픈소스 - 빠른 업데이트 그러나 버그
      - 세부적인 ACL, AuthZ를 지원

    - 개발자들에게 배포가 정확히 돌아가는지 검증을 받음 - 매번 과제별로 확인 받는게 어려움 > 이 부분을 Slack으로 자동화 - 의사결정(judgement)받음

    - Pipeline as Code
      - JSON형식의 Spinnaker 파이프라인 스크립트 - Jenkins와 Git을 통해 관리 : pipeline template를 통해 중복 스크립트 지원
      - 코드화 하여 최신 파이프라인 유지

    - Insight 획득
      - 서버그룹의 상태를 한 눈에 파악, 배포 실패 및 재해 복구를 위해 백업그룹 관리, 이를 이용해 간단한 조작으로 롤백 가능

## 클라우드 보안
- 돈과 평판
- 클라우드는 보안이 안좋다는 인식, 그러나 생각보다 보안이 좋다
- 가시성이 높아지면서 보안 개선됨
- 포괄적인 관점으로 보안을 지켜보고 있음 (서버가 아니라 랩탑에서 문제가 있을수도 있으므로)
* 인프라, 데이타, 사람
  - Beyond corp(?)
  - Istio와 같은 서비스 메쉬? 네트워크를 오픈소스화, 서비스 to 서비스
  - 스토리지와 컴퓨팅을 분리
  - IAM, 다양한 인증

* 모든 시스템에 버그와 취약점이 있음
  - 중요한 것은 어떻게 방어하고 대응하느냐 임, 취약점이 있더라도 전체에 위협이 되지 않도록..
  - 보안이 강화될수록 생산성과 편의성이 떨어짐

  - 첫번째 위협은 키를 도난 당하는 것 (피싱 같은 것)
      - 구글직원은 구글로 부터 2개의 키를 받음 - 이는 피싱에 안전

  - 서버 보드에 칩을 배치(Titan)하여 가장 낮은 레이어에서 보안 검증
      - 마더보드 등 하드웨어와, OS 는 모두 구글이 디자인/제작 - 낮은 레벨에서 신뢰확보

* 네트워크 관점
  - 자체 네트워크 시스템 구축(스위치 같은 하드웨어 포함), 자체 프라이빗 네트워크 망(세계의 25%?)으로 네트워크 보안 강화
  - 자동화된 프로세스로 안전하지 않다고 생각되면 서버 분리
  - 데이터 전송은 기본적으로 암호화(모든 단위에서도 암호화) - 암호화한 키도 암호화 - 성능보다 보안을 우선

* 인프라스트럭쳐 관점
  - 클라우드는 크기가 크지만 오히려 풀어야 할 문제가 적어 단순하다.
  - 자동업데이트, 경제적인 크기, 단일 플랫폼과 버전 > 일상적으로 챙겨야할 것이 줄어듬 > 풀어야 할 문제(데이터)에 집중 가능
  - 온프레미스, 퍼블릭 클라우드… 복잡해서 쿠버네티스 발표, 강력한 Istio 발표, 그리고 모두 인터넷에 붙어있으므로 Cloud Identity 적용하여 안전하게 공유

* 인증, 승인 관점
  - KMS 키 관리 서비스
  - HSM(Hardware Security Model) 클라우드에 키를 보관
  - 클라우드에서 최적화 하여 퍼포먼스에 영향 없음
  - Meltdown, Spectre : 인텔제안 말고 직접 패치 - 우린 안전하다 아무도 눈치 못채게 패치 완료 (비즈니스에 영향없이 언제든지 패치한다는건 굉장한 능력)
  - MFA 를 모든 서비스에 제공 - 이것으로 피싱을 막을 수 있음

* 데이터 분류/탐색
  - 데이터 보관 뿐아니라 개인정보 보호법도 지켜야 함
  - 데이터 형식은 유지하면서 값을 숨기거나 다른 값으로 매핑(실제값이 아니라 값의 종류가 중요할때 유용함)
  로깅
  - 어드민 액션은 무조건 기록되고 삭제도 못함
  - 액세스 로그는 부담 - 클라우드 서비스 제공 업체의 역할로 퍼포먼스나 스토리지에 대한 고민을 할 필요 없다
  - 접근 투명성 제공 - 구글이 고객에게 접근할 경우 반드시 공개
  - 대시보드 제공

* 컨텍스트 기반 접근 제어
  - VPC Service Control : 접근자의 조건(어디, 어떻게, 언제 등)을 판단하여 접근 허용
  - Cloud Armor : 자동으로 DDoS를 막아줌, Application attack 방어 (Web Application Firewall은 베타 중..)

* 크롬북 쨩!
  - 패치는 껏다가 켤때 사이에 적용, 사용자는 모른다. 다양한 보안 제공
  - Sandbox도 제공. Anti-Virus불필요!!

구글은 워크로드를 클라우드에 배포할때 책임의 영역을 공유해야한다고 본다. 사진의 노란색은 구글 책임, 파란색은 함께 확인해야한다.. 의견을 기다린다.
믿을 수 있는 기관으로 부터 다양한 인증을 받았다. 그래도 못 믿겠으면 직접 공격해보라 penetrace test 허용(사전 허락 불필요). 자신만만 ㅋㅋ  
cloud.google.com/security

## 마이크로 서비스 구축
- 개발자에게 인프라 권한을 넘기다보면 마이크로 서비스 구조가 됨
- 마이크로 서비스는 DevOps가 함께감
- 개발자들이 아무리 셀프서비스로 하더라도 DevOps는 플랫폼 작업을 하게됨 > CICD, 로깅 모니터링 구축
- 구글에서는 SRE가 있고.. 그러나 궁극적으로는 이들이 없어도 되게끔..
- 하드웨어 구축이 힘들다 > 가상머신 무겁고 종속성 문제 > 컨테이너
- 컨테이너는 OS위에 올리므로 가볍고, Fine Bin Packing (<> Coarse Bin Packing), 재사용성이 좋아 시간 절약
- 수 많은 컨테이너를 어떻게 관리하지? > Kubernetes 가 주류

- 컨테이너 환경으로 배포하면 개발자에게 운영 배포 권한을 줄 수 있음
- 신뢰성 효율성 성능 향상
- 보안 위협 - 도커 허브 등지 에서 이미지 가져올 경우 위험 요소가 존재
- 구글에서 이미지를 올리면 시큐리티 검증 돌려 줌
- (베이스 이미지에 시큐리티 패치 필요)
- GKE가 해주겠다! 관리형 서비스

* GKE
  - Zonal, Regional Cluster 제공
  - CDN, IAP, Cloud Armor와 연동 됨
  - 멀티 클러스터 로드밸런싱 가능
  - Stackdriver로 모니터링 가능 (기존 prometheus 같은걸로 자주 바뀌는 문제)
  - On-Prem도 가능(클러스터는 로컬에 있지만 콘솔은 퍼블릭과 통합됨)

* Shake Media 사례
  - 컨테이너기반으로 2014년부터 시작, 대부분의 워크로드를 GKE에 올림
  - 구글의 관리형 서비스를 활용하여 운영 부담 절감 - 빠른 성장에 적합
  - CICD도 관리형으로 활용 - Dockerfile이나 kubectl 사용 가능
  - 자동 취약점 체크 
  - Identity Aware Proxy : IAP - G Suite 사용자에게 VPN없이 브라우저로 공개제어

* Beyond Container : Kubernetes 다음에는?
  - 눈송이 서버 : 모양이 모두 다르다, 마치 네 서버들 처럼
  - 수정하지마라 죽이고 새로 만들어라
  - 매번 이미지를 baking해서 배포 - 이런 배포 패턴을 간단하게! Spinnaker (넷플릭스 놈들이 만들었지만, 구글도 컨트리뷰트, 아마존도)!

  - Red/Black(Blue Green) - Rolling Red Black, Canary
  - 파이프라인 구축
  - Execution Window, Manual Judgement, Manual Rollback. 기능 제공

- 규모가 커질 수록 서비스간의 종속성 파악이 어려움, 반대로 누가 원인인지도 알기 어려움
- 지능형 라우팅
- 서비스 안정성
- 보안과 정책
- 모니터링

* 노이즈 네이버후드?(장애 전파) - 서킷 브레이커 필요(장애전파 차단)
  - 해당 기능은 특정 언어에 종속된 경우가 많음 - 인프라 단으로 내리자!  Envoy proxy
  - 엄청 많은 마이크로 서비스 모두에 envoy를 설치하기 힘들다
    - Istio 사이드카 패턴, 배포할때 꼽사리로 같이 배포
  - Pilot 검색 및 데이터를 프록시에 구성
  - Mixer 정책확인, 원격 분석(우선순위 조절 등등)
  - Citadel TSL인증서를 프록시로 전송 - 모든 구간을 암호화
  - 모든구간을 모니터링하므로 여러가지가 가능 - 카나리 배포, 또는 헤더에 맞는 선택형 라우팅(트래픽 스티어링)

* 추가 컴포넌트
  - Knative - 컨테이너의 네트워크/보안 설정 등을 지원(이거는 그냥 쿠버네티스 같은데…)

* 컨테이너 모니터링
  - 계층이 다양함(하드웨어, 컨테이너, 애플리케이션, 쿠버네티스 자체)
  - 모니터링 비용 및 업무자체가 부담
  - Stackdriver - Monitor, Logging, Trace, Error Reporting, Debug
  - 다른 클라우드의 로그도 수집, 로그량에 따라 비용(필터 또는 익스포트로 절약가능)
  - 프로메테우스(쿠버네티스 모니터링) 지원 - Stackdriver로 저장하지만 외부에는 프로메테우스 형식으로 제공가능
  - 서비스 종속성 - Istio와 프로메테우스 조합으로…


## SRE
- 엔드유저의 가용성을 고려해야함
* SRE 란?
 - 일종의 소프트웨어의 신뢰성에 대해 생각하는 사고방식
   - 총 비용의 4~90%가 런치 후에 발생하지만, 개발자는 개발에만 고려
   - 민첩성(개발자) vs 안정성(운영자) > 균형점을 찾아야 함 DevOps?
 - DevOps는 조직간 사일로 타파를 위해 문화와 가이드라인 제공
   - 사일로를 줄여 커뮤니케이션 향상
   - 오류를 일상화
   - 점진적 변화 구현
   - 도구/자동화
   - 모든 것을 측정
 - 이것들과 SRE의 연관성
   - 데이터 기반
   - 운영을 소스 코드화
   - 신뢰성을 기본으로 깔고 개발
* SRE 는...
  - 개발팀과 함께하며, 확장성/신뢰성/효율성을 갖추도록 설계, 구축, 실행 솔루션 개발
  - 훌륭한 DevOps - 실천방안을 실제로 적용, 활력을 불어넣기
  - DevOps는 인터페이스, SRE는 구현?
* 주요원칙
  - 오류 예산 (핵심)
    - 가용성 = 정상시간 / 총시간 ?? (아무도 안쓰는 것이라면?, 잘못된 응답이라면?)
    - 가용성 = 정상 상호작용 / 총 상호작용 
    - 신뢰성 수준을 어느정도로 해야하는가를 결정 (90? 99? 99.9? 99.999?)
    - 신뢰도를 조금 높일 수록 다운 타임이 확 줄어든다 - 비용이 많이들고 어렵다
    - 이때 다운 타임을 오류 비율(이런 오류가 얼마나 발생할지?)로 봤을때 허용기간이 계산되어 비용이 산정가능
    - ex) 99.9% -> 26분/월 -> 오류비율 10% -> 월 36시간
    - 0%는 없다. 균형점 찾아라
    - 가용성의 반대 - 오류예산 (용인할 수 있는 다운타임)
    - 모니터링을 통해 오류예산을 소모하여 목표를 명확히 할 수 있음
      - 개발 및 SRE의 공통적 인센티브
      - 개발팀 스스로 위험 관리 / 자체 정책 마련 > 신규 피쳐 추가와 안정적 운영을 고민하게 됨
      - 비현실적인 목표는 오히려 혁신 속도를 약화
      - 시스템 가동의 공동 책임
* 측정방식
  - SLI (Indicate) 측정 지표 (성공의 척도)
  - SLO (Objective) SLI+목적
  - SLA (Agreement) SLO+결과 : 계약을 못맞추면 배상
  - 업체내에서는 SLO로 관리 - 새로운 피쳐 추가 기준
  - SLO는 합리적으로.. 무리하지말고, 기간을 두고 경험치를 기준으로
* 실행방식
  - 모니터링과 경고(관찰가능성?) : 측정 - 상황이 감지되면 경고(페이지:사람개입, 티켓:향후 조치) - SLO가 위협되면 사람이 개입
  - 수요예측과 용량 계획 : 유기적 성장계획, 비유기적(마캐팅.기능추가) 성장 결정, 신뢰성 목표 달성을 위한 여유공간
  - 초과근무 - 설계방법의 길잡이 / 모든것을 자동화불가능 / 자동화 대상 연구
  - 개인마다 하는 작업의 임계치 이상은 하지 않게끔
  - 팀 스킬 - 우수한 소프트웨어 엔지니어나 시스템 엔지니어를 채용 (50:50), 단 코딩 능력은 필요 - 개선할 스킬이 필요
  - SRE 권한 부여 - 오류예산/초과근무 예산 집행 권한 필요, SRE의 시간을 현명하게 활용, 과중한 운영부담X(팀건강 유지)
* 시작하는법
  1. SLO바탕으로 시작
  2. 소프트웨어 개발자 고용
  3. 개발/엔지니어링 조직의 여타 부문과 동등한 존중관계 유지
  4. 자율규제를 위한 피드백 경로 제공
  - 하나의 서비스 선택(한방에 하지말고 하나씩), 경영진의 지원과 후원, 문화/심리적 안정, 사례연구 수행 후 기법/지식 공유
* 도움얻는법
  - 오류예산 설정 및 준수에 경영진의 지원, 엔지니어링 투자
  - 구글은 특정 시스템이 아닌 고객의 프로덕션 시스템에 초점(CRE)
  - 구글이 지원해주는 듯
  - PSO 컨설팅 - SRE기본사항 컨설팅
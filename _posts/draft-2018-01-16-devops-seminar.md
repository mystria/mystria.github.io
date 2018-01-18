---
layout: post
title:  "내부용 DevOps 세미나"
date:   2018-01-23 13:30:00 +0900
categories: DevOps
comments: true
---
# 내부 공유용 DevOps 세미나
본 포스팅은 본인이 일하는 조직 내부에서 DevOps에 대한 이해를 넓히고자 작성한 세미나 자료입니다.
1. DevOps란?
2. DevOps 도구들
2. CI/CD
3. Monitoring
4. AWS - Resource management
5. Infrastructure as Code
6. Miscellaneous

## What is DevOps?
* Agile
  + 소프트웨어 개발 속도 향상 - 비즈니스 환경 변화에 따른 요구
  + 개발 프로세스 : Waterfall -> Agile, 최소 기능 제품(MVP) 출시 후 시장의 피드백에 대응하며 개발
* Developers vs Operators
  + 새로운 기능 개발 vs 안정적 운영 = 빠른 개발 사이클 vs 적은 변화
  + DevOps의 탄생 : 개발자와 운영자의 상호협력하여 비즈니스 가치를 실현하는 개념
  + DevOps 활동 : CLAMS(Culture, Lean, Automation, Measurement, Sharing)
* Culture, Tools, Organization
  + 조직의 문화 : 상호 존중, 신뢰, 실패에 대한 건전한 태도, 비난하지 않기
  + 도구 : 자동화된 인프라, 버전 관리 시스템, 원스텝 빌드와 배포, Feature Flag(설정값에 따라 기능 활성화를 제어), 지표 공유, 메신저 봇
  + 조직
    - 주의 : 방법이나 도구는 목적이 아님
    - 개발팀이나 운영팀만의 과제가 아닌, 전사적인 해결이 필요 - 조직내 활동 + 구조조정, 인사 지원
    - DevOps의 적용도 Agile로..

## 도구의 도입
* 다른 요소와 달리 현장의 판단으로 빠르게 적용 가능
* 필요한 도구를 체계적으로 도입
  + 전체 그림, 목표의 명확화 -> 현재 상태 및 부족한 점 분석 -> 도입 계획 수립과 단계적 실행 -> 목표에 도달 여부 확인, 새로운 과제 찾기 -> 반복
* 도구 도입 순서 : 도구들은 상호 의존성을 가짐
  1. 버전 관리 도구 : 기본 중의 기본
  2. 프로젝트 관리 도구 : 문제 공유, 일정 관리, 변경 추적
  3. 테스트 자동화 : 품질, 개발 속도 향상
  4. CI/CD : 상시 빌드로 테스트 및 문제 검출, 반드시 팀원들에게 통일된 사용법 교육 - 항상 빌드 성공 상태로 유지할 것!
  5. 가상 환경 구축 도구 : 동일한 개발 환경 공유
  6. 프로비저닝 도구 : 환경 구축 자동화, 도구 도입 후엔 무조건 도구를 사용해야 함!
  7. 모니터링 : 장애 확인, 개선과제 도출, 의미있는 지표를 정의 할 것!
* 도구 도입 후 모습
  + 커뮤니케이션 도구를 사용하여 관련자간 의사소통 활성화 : Confluence, Teams ..
  + 버전관리 시스템으로 코드 및 설정 정보 관리 : Github, Perforce ..
  + 과제, 오류, 작업을 프로젝트 관리 도구로 관리 : Jira, Github Issue ..
  + 테스트 자동화 : Gradle, MVN, TestNG ..
  + CI/CD : Jenkins ..
  + 가상화 도구를 통한 개발 및 검증 환경 통일 : Docker(Docker Compose) ..
  + 환경 구축 자동화 : CloudFormation, Chef ..
  + 운영환경 모니터링으로 문제 발견 및 프로세스 개선 : CloudWatch, ELK, Nagios/Munin ..
  + 지표 집약한 대시보드 관리 : ??
* 주의사항
  + 우선 순위를 두고 필요할 때 도구를 도입
  + 널리 쓰이는 도구(De Facto Standard)를 선택
  + 단계적 도입, 교육 및 정착 확인 필수
  + 스스로 유지 보수


## CI/CD
* Continuous Integration
* Continuous Delivery
* Continuous Deployment
* CI/CD and Tool-chain
  + GHE + Jenkins
  + Jenkins Pipeline
  + CloudFormation
  + To-Be..  
* Mindset

## Monitoring
* Infrastructure, Application, Business
* Incident Tracking
* Life-cycle of DevOps

## AWS - Resource management
* Understanding of AWS
* Other Cloud Platforms
* Resource/Service of AWS
  + IaaS
  + PaaS
  + SaaS

## Infrastructure as Code
* Dynamic Infrastructure : 동적 인프라, 자유롭게 생성/삭제 할 수 있고 바꿀 수 있음
* Immutable Infrastructure : 처음 설정한 인프라 상태 그대로를 유지
* Infrastructure Definition Tools : 필요한 인프라를 멱등성이 있는 코드로 정의
* Server Configuration Tools : 서버 관리의 자동화
* Consistency : 인프라/서버가 변경되더라도 서비스가 멈추거나 변경되어서는 안됨

## 참고자료
* 서버/인프라 엔지니어를 위한 DevOps / Jpub / 요시바 류타로 외

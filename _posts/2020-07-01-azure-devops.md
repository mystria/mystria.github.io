---
layout: post
title:  "Azure DevOps (AZ-400)"
date:   2020-07-01 09:00:00 +0900
categories: Azure
comments: true
---

# Azure DevOps
Azure Cloud 기반 DevOps 적용 방법(Azure DevOps 도구 사용법)  

## 사전 작업
* Visual Studio (Community) 설치(Windows 10, Mac, Linux 다 OK, But Windows 7은 지양)
* outlook.com 계정 생성, dev.azure.com을 가입 후 조직 생성
  + .NET Core(cross platform에 적합), .NET Framework(이런저런 사정으로 윈도우 전용)
  + 미래에는 OS의 구분이 불필요해지는 방향으로 발전 중
* Visual Studio Code 설치, Dotnet 동작 확인
  + $ dotnet new mvc
  + $ dotnet restore
  + $ dotnet build
  + $ dotnet run
* Git 설치
  + IaC를 적용하기에 필요 요소(인프라를 코드로 정의 -> Git에서 버전관리) : Azure ARM(Azure Resource Management?) 
  + git clone https://github.com/Microsoft/azuredevopslabs/

## 학습자료
* DOCS
* Learning : https://docs.microsoft.com/ko-kr/learn
* YouTube : https://www.youtube.com/user/microsoftlearning 
* GitHub : https://github.com/MicrosoftLearning (Hands-on Lab 자료가 들어있음)
* DevOps : https://azuredevopslabs.com

## 자격시험 팁
* 시험장 vs 집 : 집은 proctor가 깐깐하게 봄
* 영어 vs 한글 : 한글 시험은 영어로 toggle가능 - 한글 지문을 읽고, 애매한 부분은 영어로 확인 (답 기입시엔 한글로 돌아와야 함)
* 기출 문제 학습이 도움이 됨

## Cloud 배경 지식
* Netflix, Uber, Skype, Amazon .. 모두 소프트웨어 회사
* JP Morgan, Walmart, GE, Ford .. Cloud기반 소프트웨어 회사가 되어감
* IaaS(자차), PaaS(랜트), SaaS(택시)
  + Cloud는 PaaS에 가까움
  + DevOps도 PaaS에 더 가까움 
  + PaaS: 클라우드에서 제공되는 완전한 개발 및 배포 환경, 완전한 응용 프로그램 수명 주기(빌드, 테스트, 배포, 관리, 업데이트) 지원
    - 대부분의 PaaS사용자는 IaaS도 같이 사용: 왜? legacy..
    - Public cloud와 private cloud를 같이 사용 - hybrid
      (현재 트랜드는 한 곳 몰빵이 아닌 multi cloud로 fail over구성)
    - Azure는 이러한 고객 요구사항에 잘 대응 

## DevOps 개요
* 지속적으로 가치를 전다라기 위한 사람, 프로세스, 도구의 집합체
* Waterfall vs Agile (현 개발업계의 75%가 agile을 도입) 
  + 한 번 완벽한것 보다, 자주 고치자
  + 자주 고치고 배포하려면? 자동화 필요 - CICD
* 도구 이전에 문화
  + 개발(Dev)과 운영(Ops)을 한방에
  + 반복은 자동화
  + 소통이 최상의 방법론
* 최근에는 DevOps의 필요성에 대해서는 공감대가 형성
  - 배포주기가 빨라짐
* Azure DevOps
  + 단계별 프로세스
    - Azure Board: 요구사항 추적 관리(Jira)
    - Repository: 코드 저장소
    - Pipeline: CICD
    - Test Plans: 테스트 시나리오 저장
    - Artifacts: 패키지 관리
  + 한 스프린트 
    - 작업(요청) 내역 -> 개발 -> 코드 저장소 -> 빌드 -> 테스트 -> 배포(Stage) -> 배포(Prod) -> 피드백
    - Board -> Visual Studio -> Azure Repo -> Build Agent, Azure Test -> Pipeline -> Feedback client
  + 실습 
    - (프로젝트 기획 -> )게시판(board)생성, 작업자배정, 작업요구내역서 -> 웹앱생성 -> 코드,브랜치관리 -> 빌드/테스트 -> 배포 -> 앱 인사이트, 피드백
  + 장점
    - 왠만하면 무료, WBS등 레거시와 연동, 커스터마이즈
    - 다른 repository와 연동
    - Test 프로젝트쌍으로 말아두면, 알아서 수행하고 결과 보고
    - Teams, Slack 등과 연동
    - AKS와 연동
* 실무 대응
  + 요구사항 관리WBS(vs Workitem) 작성, Dictionaly 작성 - Jira와 매우 유사함(sprint관리, kanban보드, query편집)
    - xls파일로 관리 가능 (Team plugin)
    - MS Project와 연동(deprecated권장, Boards가 완벽 커버)
    - Visual Studio, Eclipse에서 접근 가능
  + 개발 + 테스트: 개발, 테스트
    - 코드를 repository에 commit 할 때, #{task번호} 를 commit message에 포함하면 자동으로 task link에 연결됨
    - 테스트(정적분석, 커버리지, 동적분석, 논리분석 등)로 코드 품질 보증, 개발 사이클 동안 반복 수행, 한 번 잘 해두면 두고두고 좋음
    - Azure DevOps에서 제공하는 성능/부하 테스트
  + 배포(Pipeline(대메뉴))
    - 자동화된 기능 테스트 - 통홥 - 빌드
    - Visual Studio에서 바로 배포 환경(바로 WebApp, 아님 Pipeline)도 구성 가능
    - Pipeline(소메뉴, 사실상 CI) 후 Release(소메뉴, 사실상 CD) 로 분리된 단계
    - Pipeline
      - 빌드 파이프라인, Tasks와 Triggers(Git에 hook)설정
      - 소스, 빌드환경 구성, 빌드, 테스트, 아티펙트 저장
      - Agent pool(& specification): 빌드를 수행할 환경, 무료(Azure Pipeline)을 선택을 하면 빌드자원을 할당받아 처리하기 때문에 다소 느림, 유료의 사설 빌드자원을 생성할 수 있음
      - 대부분의 Git이 연동이 되지만, git의 event를 수신할 수 있는 네트워크 환경이어야 함
    - Release
      - (CI가 제대로 생성, 한 번 이상 수행되어야 함, 하지 않으면 CD를 생성할 수 없음)
      - 배포 파이프라인, Artifact(CI와 연결) 획득 이후 커스텀 할 수 있는 stage들을 구성
      - Artifact에서 build가 생성될 때 시작하도록 trigger 설정
      - 배포 단계에서 인프라도 생성하는 단계를 넣을 수도 있음
  + 모니터링 도구
    - 현업 담당자: 초기요구사항, 검토, 버그수정요구, 추가요구사항 - 적극적일수록 개선이 잘됨
    - Application Insight: 매소드/프로퍼티 단위 입출입까지 수집 가능, Visual Studio에서 의존성 추가 후 바로 리소스 생성 가능
  + 피드백
    - Browser의 plugin에 Test&Feedback 기능 설치
    - https://dev.azure.com/계정 에 연결 시, board 선택 가능
    - 화면 캡쳐, 녹화, 메시지 등을 수행 후 board에 task나 bug로 등록 가능
    - Boards에서 자연스럽게 처리

* 개발도구 관련
  + Visual Studio
    - 동적인 유닛테스트 지원
    - 코딩된 UI테스트 지원(Xamarian, 모바일/웹 UI 테스팅 팜)
    - WBS와 코드 연동
    - DB 스키마 버전 관리
  + Azure DevOps(Visual Studio)
    - 성능/부하 테스트 지원(WebAndLoadTest, Cloud/OnPrem 환경에서 가상의 부하 발생)
    - Git 접근만 제공해주기 위한 Personal Access Tokens
  + 소스 제어
    - 중앙집중식 소스 제어에서 분산식으로 변화
    - Azure: Git(분산) vs TFVC(중앙집중) 둘 다 지원
    - TFVC를 Git으로 마이그레이션(git-tfs)
    - Azure인프라 설정도 ARM(Azure Resource Management)라는 json 형식의 코드로 관리, 기존의 절차형 인프라 관리에 비해 공유, 버전관리 가능
    - 다른 repository를 import 가능, git fork(원 repo와 연결 유지)와 clone의 차이? 
    - 멀티 레포지토리 가능: 원활한 코드관리, 테스트 등..
    - 워크플로우 전략: Feature branching / Gitflow branching / Forking Workflow
  + Visual Studio Code
    - Git ignore 확장 사용
    - 실습: [Git실습](https://www.azuredevoposlabs.com/labs/azuredevops/git/) | [CI실습](https://www.azuredevoposlabs.com/labs/azuredevops/continuousintegration/)
  + Wiki
    - 개발 산출물(설계문서 등)을 위키로 관리, Azure DevOps에서 위키도 기본으로 제공
    - 흩어져서 관리하지 말고, Azure DevOps로 일원화 하여 형상관리 하기를 권장
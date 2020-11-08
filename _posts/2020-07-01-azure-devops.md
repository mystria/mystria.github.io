---
layout: post
title:  "Azure DevOps (AZ-400)"
date:   2020-07-01 09:00:00 +0900
categories: Azure
comments: true
---

# Azure DevOps
Azure Cloud 기반 DevOps 적용 방법(Azure DevOps 도구 사용법)  
그 외 DevOps 개념 및 도구에 대한 설명

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
* 배포 전략(배포 다운타임 최소, 신속한 롤백)
  + Blue Green: 동일한 환경 보장으로 최소 하나의 운영을 보장(라우터로 구분), 
  + Canary: 일부에게 미리 배포 하며 확인 후 전체 배포, 여러 버전이 동시에 운영되는 리스크
  + Azure WebApp에서 다양한 배포전략 대응해 줌
* 컨테이너 서비스
  + 컨테이너는 작고-저렴하고-빠르고-격리된 서비스, Docker는 컨테이너를 이미지로 관리하는 플랫폼, K8s는 Docker 오케스트레이션
  + 최근 클라우드기반 솔루션 시장에서 대세 기술
  + 이 Docker 기술이 agile, CI/CD, DevOps와 궁합이 좋음
  + K8s 트래픽 진입점(LoadBalancer, Ingress)
    - Ingress로 Micro Service Architecture 구현(path를 기준으로 service 연결, API Gateway 유사)
* 보안
  + 팀구성원 전체의 책임
  + 인프라 - 앱 아키텍처 - 지속적인 검증 - 모니터링
  + SQL Injection 공격 (데이터 탈취 후 추적을 막기위해 파괴)
  + 컴플라이언스(규정준수) 개발 프로세스 구현: 정적분석 - 테스트, 취약점 검사 - 패시브 펜 테스트, 인프라 검사, 회귀 테스트
    - 인프라 취약성 검증: Azure Security Center, Azure Policy, 인프라 유효성 검사
    - CI단계에서 검증: Azure DevOps Pipeline과 연계가 잘 됨(Checkmarx, BinSkim, BlackDog? 등) 
    - CD단계에서 검증: OWASP ZAP 침투 테스트
    - 아키텍처 검증: Microsoft Threat Modeling 도구 제공
* 구성 정보 관리
  + 구성 보유자 / 구성 소비자 / 구성 저장소 / 비밀 저장소 로 구분
  + KeyVault 제공: 표준 암호화, 하드웨어 보안으로 구성, Pipeline과 연계 가능 
    - 중앙집중식, 액세스 및 사용 모니터링 
    - 접근 범위(Public, Private등) 설정
    - 실습: [Azure KeyVault](https://www.azuredevopslabs.com/labs/vstsextend/azurekeyvault/)
    - K8s의 Secret 대용으로 적용 가능
    - Dev(개발자)는 외부에 존재하는 개발용 구성 사용, Ops에서 구성 관리
* 코드 퀄리티
  + SonarQube: 코드품질 향상 플랫폼, Azure DevOps와 연동 - SonarCloud
  + WhiteSource: 오픈 소스의 보안 및 라이선스 체크 - WhiteSource Bolt
  + 개발 시 수정하는게 배포 후 수정하는 것 보다 수백배 저렴, DevSecOps
* 모바일 DevOps
  + 일반적인 DevOps와 다르진 않으나 Visual Studio App Center 로 관리
  + 모바일 앱 개발과 배포, 테스트에 특화

## Azure DevOps feature 활용(실습)
* Project 생성, Public/Private 지정 가능 
* 요구사항 관리WBS(vs Workitem) 작성, Dictionaly 작성 - Jira와 매우 유사함(sprint관리, kanban보드, query편집)
  + xls파일로 관리 가능 (Team plugin)
  + MS Project와 연동(deprecated권장, Boards가 완벽 커버)
  + Visual Studio, Eclipse에서 접근 가능
* 개발 + 테스트: 개발, 테스트
  + 코드를 repository에 commit 할 때, #{task번호} 를 commit message에 포함하면 자동으로 task link에 연결됨
  + 테스트(정적분석, 커버리지, 동적분석, 논리분석 등)로 코드 품질 보증, 개발 사이클 동안 반복 수행, 한 번 잘 해두면 두고두고 좋음
  + Azure DevOps에서 제공하는 성능/부하 테스트
* 배포(Pipeline(대메뉴))
  + 자동화된 기능 테스트 - 통홥 - 빌드
  + Visual Studio에서 바로 배포 환경(바로 WebApp, 아님 Pipeline)도 구성 가능
  + Pipeline(소메뉴, 사실상 CI) 후 Release(소메뉴, 사실상 CD) 로 분리된 단계
  + yml로 파이프라인 정의 가능(실제로 코드로 저장됨, 수정할 때 마다 커밋 메시지 입력)
  + Pipeline(예전이름 Build)
    - 빌드 파이프라인, Tasks와 Triggers(Git에 hook)설정
    - 소스, 빌드환경 구성, 빌드, 테스트, 아티펙트 저장
    - 대부분의 Git이 연동이 되지만, git의 event를 수신할 수 있는 네트워크 환경이어야 함
    - Agent pool(& specification): 빌드를 수행할 환경, Azure Pipeline(무료)을 선택을 하면 공용 빌드자원을 할당받아 처리하기 때문에 다소 느림, 사설 빌드자원(비용발생)을 생성할 수 있음, *반드시 하나 이상의 에이전트가 필요*하며 pool 단위로 build order를 수행
      - Agent pool은 물리적으로 여러 컴퓨터에 존재하는(한 컴퓨터에 여러개도 가능한듯) 에이전트를 논리적으로 묶어 구성할 수 있음, 여러 프로젝트에서 공유가능(Project settings에서 existing 추가해주면 됨)
      - MS 호스팅 에이전트: 무료/유료 옵션, 무료도 성능 충분(단, 시간 제한이 있음, 무료는 병렬 안됨(병렬 설정 시엔 큐에 대기))
      - 자체 호스팅 에이전트: 내부망 등에 붙어야하거나 커스터마이즈가 필요한 경우, Azure Pipeline은 이를 오케스트레이션, 병렬 작업 지원, 호스팅할 서버(또는 PC)에 agent를 설치 후 설정(해당 서버에 빌드환경이 준비되어 있거나, 빌드스크립트를 통해 빌드환경을 준비하도록 해야함)
  + Release
    - (CI가 제대로 생성, 한 번 이상 수행되어야 함, 하지 않으면 CD를 생성할 수 없음)
    - 배포 파이프라인, Artifact(CI와 연결) 획득 이후 커스텀 할 수 있는 stage들을 구성
    - Artifact에서 build가 생성될 때 시작하도록 trigger 설정
    - 배포 단계에서 인프라도 생성하는 단계를 넣을 수도 있음
* Pull Request
  + CI 전, 코드 리뷰 및 토론의 기회
  + Complete 할 때 merge 방식 지정 가능
  + Branches 화면에서 branch별 policy를 설정하여 Pull request 정책(필수 리뷰어 수, WorkItem link 등)을 강제 가능
  + PR의 WorkItem link는 commit의 #link와 다른 개념, PR은 WI를 close 할 수도 있음
  + 실습: [PR실습](https://www.azuredevopslabs.com/labs/azuredevops/pullrequest/)
* 모니터링 도구
  + 현업 담당자: 초기요구사항, 검토, 버그수정요구, 추가요구사항 - 적극적일수록 개선이 잘됨
  + Application Insight: 매소드/프로퍼티 단위 입출입까지 수집 가능, Visual Studio에서 의존성 추가 후 바로 리소스 생성 가능
* 피드백
  + Browser의 plugin에 Test&Feedback 기능 설치
  + https://dev.azure.com/계정 에 연결 시, board 선택 가능
  + 화면 캡쳐, 녹화, 메시지 등을 수행 후 board에 task나 bug로 등록 가능
  + Boards에서 자연스럽게 처리
* 패키지 배포
  + 실습: [Nuget Push](https://www.azuredevopslabs.com/labs/azuredevops/packagemanagement/)
  + Artifacts > Feed를 생성하면 사설 NuGet(의존성 저장소)이 만들어짐
  + 라이브러리 프로젝트 빌드 후 패키징(pack)을 하고, 이 저장소에 push하면 라이브러리가 업로드 됨
  + 해당 NuGet을 프로젝트에 추가하면 라이브러리를 의존성에 추가할 수 있음
  + 위 과정 또한 pipeline으로 구성(코드 커밋 시, 빌드-패키징 하고 이를 release에서 nuget push하면 됨)가능
  + NuGet에 업로드된 목록이 최신 상태로 업데이트 되는 시간이 다소 필요함
* 배포전략
  + WebApp의 slot을 이용하여 배포전략 적용이 간편함
  + Staging을 production으로 올릴 때, production에 새버전을 배포하는 것이 아니라 swap(교환)을 통해 blue-green을 교환할 수 있음
  + Release의 이력을 재실행해 더 예전 버전을 다시 배포할 수도 있음
* 컨테이너 서비스
  + Portal에서 AKS, ACR 등 인프라 생성, Deployment center(미리보기)에서 마법사로 CI/CD 파이프라인 구성 / 또는 수동 구성(사전정의된 구성이 없으므로 Empty job으로 생성후 Docker task 추가)
  + ACR에 푸시된 이미지는 portal의 ACR의 repository 탭에서 확인 가능
  + Secret 정보를 Parameter로 관리 -> pipeline에서 Secret resource로 생성 -> manifest로 배포시 secret값을 $()변수로 전달
  + 자동으로 모든 resource에 namespace가 할당되는 이슈가 있네..
* 코드 퀄리티
  + VS 마켓플레이스(Azure DevOps탭)에서 Azure DevOps의 Org와 연결 필요
  + [SonarCloud](https://sonarcloud.io/account/security/)
    - 마켓플레이스에서 org와 연결했다면 pipeline에 template 등록됨
    - SonarCloud에서 발급한 키로 pipeline과 연동
    - Pipeline에서 빌드 후 코드 검사 실행, 실행결과를 SonarCloud로 리포팅
    - 실습: [SonarCloud](https://www.azuredevopslabs.com/labs/vstsextend/sonarcloud/) 
  + WhiteSource Bolt
    - 마켓플레이스에서 org와 연결했다면 Pipeline(대메뉴)에 메뉴가 추가됨
    - 설정 후 실행하면 해당 메뉴에서 바로 결과 확인 가능
    - 실습: [WhiteSource](https://www.azuredevopslabs.com/labs/vstsextend/whitesource/)
* 모바일 DevOps
  + [AppCenter](https://appcenter.ms)에 App 추가
    - 배포 그룹(Distribute Group) 생성, 유저 추가
    - Visual Studio의 Xamarin.Form 프로젝트로 모바일 앱 개발 환경 구축
    - 로컬에서 빌드, 서명 필요(App Store, Google Play, 사설 등), 아카이빙(보관)
    - 아카이빙 된 Mobile App(apk파일) 을 배포그룹에 업로드 -> 배포그룹 유저들에게 안내됨
  + 이 과정을 자동화
    - Azure DevOps는 크게 사용하지 않음, Git repository 용도로만 사용
    - 코드 Git에 푸시
    - AppCenter에 Build메뉴에 repository 등록 후 파이프라인 설정

## 개발도구 관련
* Visual Studio
  + 동적인 유닛테스트 지원
  + 코딩된 UI테스트 지원(Xamarian, 모바일/웹 UI 테스팅 팜)
  + WBS와 코드 연동
  + DB 스키마 버전 관리
* Azure DevOps(Visual Studio)
  + 성능/부하 테스트 지원(WebAndLoadTest, Cloud/OnPrem 환경에서 가상의 부하 발생)
  + Git 접근만 제공해주기 위한 Personal Access Tokens
  + 오픈소스 프로젝트에 무료(확인필요)로 제공(사용자, 파이프라인 수 제한은?)
  + Pipeline을 yml로 정의가능, UI 디자이너 제공
    - Stages > Jobs 로 단계 구성, depensOn으로 단계 순서 강제가능(여러개를 의존하여 병렬 처리 가능)
    - Environment를 이용해 결제 프로세스도 구성 가능
    - Multi-stage 지원: 한 파일에서 여러 단계를 정의
* 소스 제어
  + 중앙집중식 소스 제어에서 분산식으로 변화
  + Azure: Git(분산) vs TFVC(중앙집중) 둘 다 지원
  + TFVC를 Git으로 마이그레이션(git-tfs)
  + Azure인프라 설정도 ARM(Azure Resource Management)라는 json 형식의 코드로 관리, 기존의 절차형 인프라 관리에 비해 공유, 버전관리 가능
  + 다른 repository를 import 가능, git fork(원 repo와 연결 유지)와 clone 
  + 멀티 레포지토리 가능: 원활한 코드관리, 테스트 등..
  + 워크플로우 전략: Feature branching / Gitflow branching / Forking Workflow
* Visual Studio Code
  + Git ignore 확장 사용
  + 실습: [Git실습](https://www.azuredevopslabs.com/labs/azuredevops/git/) / [CI실습](https://www.azuredevopslabs.com/labs/azuredevops/continuousintegration/)
* Wiki
  + 개발 산출물(설계문서 등)을 위키로 관리, Azure DevOps에서 위키도 기본으로 제공
  + 흩어져서 관리하지 말고, Azure DevOps로 일원화 하여 형상관리 하기를 권장
* Jenkins
  - 완전히 설정된 Jenkins 이미지를 바로 Azure에 배포가능
  - CI, CD 모두 지원하는 범용성 높은 도구
  - 실습: [Jenkins Build](https://www.azuredevopslabs.com/labs/vstsextend/jenkins) / [Tomcat 베포](https://www.azuredevopslabs.com/labs/vstsextend/tomcat/)
  - Azure Pipeline과 연계
    - Jenkins admin의 API Key필요
    - 우선 Jenkins에 build job 생성, source는 Git으로 부터..
    - 접근1: (Project settings)Service hooks: Code push -> Jenkins Url: 접근1은 Azure가 Jenkins를 호출하는 것
    - 접근2: (Project settings)Service connection에 Jenkins추가 -> Pipeline(connection 추가, sync 비활성화)의 task를 Jenkins로 추가 -> 산출물을 받아와 drop에 저장: 접근2는 빌드 작업만 Jenkins를 활용하는 것
    - Jenkins의 결과를 받아 CD(Release) 호출 가능
* AKS(Azure Kubernetes Service) 
  - Azure에서 K8s를 쓰기위한 가장 쉬운 방법 
  - ACR(Azure Container Registry): Docker image를 사설로 관리하는 서비스
  - AKS를 resource group에 추가하면 별도의 RG가 생성되어 K8s자원을 관리(사용자가 알 필요는 없음)
  - Cloud Shell로 쉽게 접근/제어 가능, K8s console도 활성화 가능
  - Azure Pipeline과 연계 가능
  - 실습: [Kubernetes](https://www.azuredevopslabs.com/labs/vstsextend/kubernetes) 
* Visual Studio 앱 센터
  + 모바일 환경에 최적화된 DevOps경험을 제공
  + 모바일 테스트 프레임워크등 다양한 도구 -> Visual Studio App Center로 통합 
  + 다양한 플랫폼(Android, iOS, Windows 등) 지원, UI 테스트 프레임워크(Xamarin, Appium..)로 자동화
  + 지속적 통합 - 테스트(분석 및 보고) - 배포(업데이트, AppStore, Google Play연동) - 사용/에러 분석
  + 배포그룹 단위로 앱을 배포(배포그룹(distribute group): AppCenter내 개발자/테스트 구성, 비공개(초대된 테스터)/공용/공유 구분)
  + 실제 테스트용 장치 프로비저닝 및 관리

## 후기 및 정리
Azure DevOps는 강력하고 다양한 기능이 있는데 각각의 연결이 다소 삐걱거림(특히 Subscription), 이를 능숙하게만 다룬다면 최고의 효용을 발휘할 수 있을 것 같음  
고객이 필요로 할만한 기능은 다 지원하며, 쉽게 관리할 수 있음 - Microsoft Windows를 쓰는 듯한 기분  
AWS DevOps(2018년 교육, 2년의 차이가 있어서 그럴 수도 있지만) 보다 더 개발자 개발환경과 연동이 잘됨  
AWS는 그냥 자체 개발한 AWS전용 DevOps 같다면, MS의 기존 개발환경과 통합을 중점적으로 한 것 같음(GitHub을 인수하여 Git중심으로 관리하는게 신의 한 수), DevOps 기능 자체의 완성도나 자유도가 훨씬 높음  
Resource Group으로 쉬운 자원 정리, Key Vault로 편리한 개발 구성 관리  
AKS는 EKS와 거의 똑같음  
Java 개발자라 그런지 Visual Studio는 좋은지 모르겠음, Azure Cloud Web Console은 다소 산만함  

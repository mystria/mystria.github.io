---
layout: post
title:  "Azure 기본 (AZ-900)"
date:   2020-05-25 12:00:00 +0900
categories: Azure
comments: true
---

# Azure 기본 교육
Azure Cloud 사용을 위한 기본 개념 및 Azure 서비스 소개

## Cloud(이하 클라우드) 개념
* 클라우드 특성: 고가용성, 확장성(글로벌), 탄력성, 민첩성(빠른 자원할당), 내결함성(보안)
* 클라우드 공급자(Azure, AWS, Google)은 규모의 경제를 통해 저렴한 인프라 제공
  + CapEx: 자본지출, 기존 인프라는 투자비용(초기비용↑, 감가상각), OpEx: 운영비용, 종량제
  + 적은 CapEx, 포기비용? zero
* 클라우드 모델
  + 퍼블릭 클라우드: Azure, AWS등.. 공급자가 리소스를 소유하고 최신 상태로 업데이트, 인터넷으로 연결(connectivity), 비교적 쉬운 관리(infra 전문지식 불필요)
  + 프라이빗 클라우드: 조직이 리소스와 운영책임을 가짐, 완벽한 리소스 제어, 조직 구성원만 접속가능(강화된 보안)
  + 하이브리드 클라우드: 둘을 합쳐서 적절한 위치에서 app 실행, 둘의 장점을 취하고자 함, 퍼블릭으로는 부족한 지역적 법/규정적 요구사항 준수가능(준수성), 비용↑, 관리↑
* 클라우드 서비스 유형(크게 3가지, 기타)
  + IaaS: 대부분의 기본 클라우드 컴퓨팅 서비스의 범주, 종량제 리소스(머신, 스토리지, 네트워크 등)인프라, 인터넷으로 프로비저닝/관리되는 인스턴트 컴퓨팅 인프라
    - 시나리오: 워크로드를 그대로 옮겨서 재현가능(클라우드로 마이그레이션 하는 기업에서 많이 적용), 빠른 자원 확장, POC 해보기 좋음, 기존 웹호스팅 보다 저렴, 백업 및 복구 단순화
    - 비유: 식당의 원재료만 공급
    - Control-ability(제어의 범위) 이 높음, 기존 워크로드를 유지하고자 하는 경우 유리, 인프라 외에는 엔지니어의 관리가 필요
  + PaaS: 소프트웨어 개발, 배포, 테스트 환경 제공, 운영체제/개발도구/데이터베이스/비즈니스 분석을 제공하여 서비스 개발에만 집중 가능하고 빠른 app 배포 가능
    - 시나리오: 스타트업
    - 비유: 반조리된 재료 공급 - 간단한 조리 후 판매
    - 제어의 범위가 줄어듬
  + SaaS
    - 시나리오: Office 365같은 서비스
    - 구독료 개념(종량제이지만 기본료가 포함), 제어의 범위가 좁음
    - 비유: 완제품
  + 기타: DaaS(데이터베이스), TaaS(테스트)
  + 요약: 높은 제어력(IaaS) vs 개발에만 집중(PaaS)

## 핵심 Azure 서비스
* Azure 아키텍처 구성요소
  + region(지역, location): 데이터 센터의 집합, 사용자와 가까운 지역, 지역별 배포 가용성이 다름, 별도의 글로벌 서비스도 존재
  + region pair(지역쌍) 존재: 고가용성을 위함, 복제 보관
  + 가용성 옵션: 단일VM, 가용성 세트, 가용성 영역
  + 가용성 세트(Set): 논리적으로 구분한 가용영역, 2가지 설정: UD(업데이트 도메인) - 업데이트 시 순차적으로 적용되도록 분리, FD(오류 도메인) - 물리적(변압기)으로 분리
  + 가용성 영역(Zone): 물리적 분리
  + 리소스 그룹: 동일한 수명주기를 가지는 리소스의 컨테이너, 지역으로 묶는게 (관리등에서)좋지만 무조건은 아님
  + Azure Resource Manager
* 서비스
  + Azure Computing Service: On-Demand 컴퓨팅 서비스, 디스크,프로세서,메모리,네트워킹,운영체제 같은 컴퓨터
  + 가상머신 서비스: IaaS, VM Scale Sets(동일한 VM의 자동크기 조정 - auto scaling)
  + App Service: PaaS
  + Function: MicroService용(서버리스?)

* 실습
  + portal.azure.com
  + CLI: az
  + 리소스 그룹 만들기: 논리적인 그룹, 그룹 자체는 무료, 서브스크립션 지정 (향후 삭제시 리소스 그룹 내부 자원들은 의존성을 고려햐여 순차 삭제됨)
  + VM 만들기: 리소스 그룹 지정, 지역, 가용성 옵션(set vs zone), 이미지 선택, 이 외에도 시스템 자원도 세부 설정 가능
  + Virtual Network: AWS의 VPC와 Subnet과 동일한듯, https://azure.microsoft.com/ko-kr/services/virtual-network/

* Azure의 데이터 범주
  + 구조적 데이터: 스키마 준수(센서 데이터, 재무 데이터)
  + 반구조적 데이터: 임시 스키마, NoSQL(json, HTML)
  + 비구조적 데이터: 스키마 또는 데이터 구조 없음(PDF, JPG, 동영상)
* Azure 스토리지 서비스
  + IaaS: 디스크, 파일
  + PaaS: 컨테이너(Blob), 테이블, 큐
    - Blob 컨테이너: 리소스 그룹, 지역, 성능(standard, premium), Replication 설정(같은 지역내에서는 트래픽 비용이 없음, 지역 외에서는 outbound에 대해 비용발생), 저렴하지만 실시간성은 조금 떨어짐
    - File share: SMB, dropbox나 iCloud같은 역할, 네트워크 설정 필요
    - 테이블: NoSQL, partitionKey - 고정 데이터, rowKey - 유니크 키
    - 큐: 몰려오는 데이터 트래픽을 쌓아뒀다가 순차 처리용, (별도로 다른 큐 서비스도 있음)
  + 스토리지 익스플로러로 쉽게 관리: https://azure.microsoft.com/en-us/features/storage-explorer/
* 네트워크
  + Network Rule: (AWS의 SG같은 거) Inbound 와 outbound로 구분, 기본적인 설정이 추가되어 생성됨
  + Public IP: 유동이지만 고정도 가능
  + Network Watcher로 topology 확인 가능
* Function
  + IaaS를 선택할 필요가 없이 바로 코드를 올릴 수 있고, docker 도 지원
  + App name은 바로 domain으로 노출됨
  + plan: Consumption(Serverless, 사용될 때만 호출, 한달에 100만번(0.1초 수행단위) 무료), App service plan(컴퓨터를 켜둔것과 같은 비용), Premium
    - Serverless: 함수간 영향이 없음, 서버 유지 비용, 고가용성 설정 비용 등이 없음, 되도록 10분 이하(5분이하 권장)로 하길(10분 정도면 Web App사용)
  + Application settings: 앱을 위한 환경변수
  + Connection strings: 해당 리소스에 접속하기 위한 보안키?
  + WebHook+API / Timer / DataProcessing(DB에 데이터가 추가되면 자동실행)
* Azure 데이터베이스 서비스
  + CosmosDB
  + SQL DB
  + Migration service
  + 등등
* IoT
  + IoT Central: SaaS, 글로벌 IoT대규모 연결, 모니터링, 관리(시각화 툴)
  + IoT Hub: 관리형 서비스, 앱과 디바이스간의 양방향 통신, 메시지 허브
  + REST는 좀 overhead가 있으니(REST는 서버에 1초에 4~5번 정도만 call 가능) MQTT/AMQP사용(1초에 70~80번 가능)
  + Stream Analysis: Stream -> (input) - IoT Hub, Event Hub(query) - (output) -> 테이블 스토리지(json)
* 빅데이터 및 분석
  + SQL Data Warehouse
  + HDInsight
  + Data Lake Analytics
* 인공지능
  + Machine Learning Service
  + Machine Learning Studio
* 서버리스
  + Function
  + Logic App
  + Event Grid
* 데브옵스
  + DevOps Service
* Azure App Service: 관리형 앱 서비스 - 코드나 docker를 그냥 올리면 됨
  + Web App
* 관리도구
  + Portal
  + CLI, PowerShell(MacOS, Linux도 지원)
  + Cloud Shell
  + 모바일 앱
  + Azure Advisor
  + Export template: 현재 설정 그대로 다른 곳에서 다시 만들 수 있게 json형식의 template을 export

## 보안
* Azure 방화벽 - 리소스 보호를 위해 IP주소 기반으로 ACL, 상태저장, 관리, 서비스용 방화벽(FaaS)
  + 고가용성, 모니터로깅
  + 비 HTTP트래픽도 관리
  + Subnet에 자원처럼 생성(IP를 가짐)
  + 규칙: NAT규칙, 네트워크 규칙, 애플리케이션 규칙
    - 네트워크 규칙: source와 대상 IP를 기준으로 port 허용 규칙
    - Application 규칙: source IP와 FQDN으로 port 허용 규칙
* WAF - Application 레벨에서 보호
* DDoS 보호
  + 원치않는 네트워크 트레픽 제거
  + 기본 Tier: Azure에서 자동으로 제공
  + 표준 Tier: 완화 기능을 추가 가능, inteligent한 기능 추가 가능
    - 볼륨 공격: overflow유도
    - 프로토콜 공격: 프로토콜 약점을 통해 공격
    - 리소스 공격: 리소스간 데이터 통신 방해
* 네트워크 보안그룹(NSG): NIC에 바인드 되어 인/아웃바운드의 ip, port(protocol) 규칙 추가
* 참고) 심층방어(defence index): 물리 - 신원 및 액세스 - 경계 - 네트워크 - 컴퓨팅 - 응용프로그램 - 데이터
  + 경계 부분에 Azure DDoS 보호나 Azure 방화벽 구축,
  + 네트워크 부분에서 네트워크 분할(서브넷)이나, NSG, 인/아웃바운드 통제
* Azure 신원확인 서비스
  + ID management: Authenticate(인증, AuthN) and Authorization(권한부여, AuthZ)
  + 인증: 사람 또는 서비스를 식별
  + 권한부여: 액세스할 수 있는 데이터/작업의 수준 결정
  + Azure Active Directory(AD) - 주의, 윈도우의 AD와는 다른 기능, 서비스임
    - 인증
    - SSO(Same Sign-On은 뭐지?)
    - 응용 프로그램 관리: AzureAD의 App등록에서 설정
    - B2B/B2C의 ID 서비스: AzureAD B2B, AzureAD B2C 두 가지를 만들 수 있음
  + Azure MFA: ID/PW외에 추가 인증요소 요구(3가지 조건: 당신이 알고 있는 것, 당신이 가지고 있는 것, 당신을 나타내는 것)
* 추가 보안 도구
  + Azure 보안 센터: Azure 및 온프레미스의 위협 보호 기능을 제공하는 모니터링 서비스
    - 보안 권장 사항 제공
    - 보안 설정 모니터링
    - 신규 리소스에 보안 정책 자동 적용
    - 사용 시나리오: 감지, 평가 및 진단 단계에서 보안 센터 활용
  + Azure Key Vault: 암호화 토큰, 키를 클라우드에 저장
    - 기밀(secret), 키, 인증서 관리 - 접근과 모니터링 제공
    - HSM(하드웨어 보안 모듈)을 지원 - premium 요금제
  + Azure 정보 보호(AIP): Office 365에서 잘 사용 중
    - 레이블을 적용하여 문서 및 전자 메일을 분류/보호
  + Azure 고급 위협 보호(Azure ATP)
    - 지능형 위협, 손상된 ID, 악의적 내부자 작업을 식별/탐지/조사, 구성요소: 포털, 센서
    - 포털: 전용 모니터링 포탈
    - 센서: 도메인 컨트롤러에 설치됨
    - 클라우드 서비스: Azure에서 실행됨

## Azure 거버넌스
* Azure 정책: 리소스에 대한 규칙과 효과를 적용하기(강제하기) 위한 것, 회사 SLA 준수
* Azure 데이터 기능: 기본 정책, 이니셔티브 또는 사용자 정책으로 준수여부 평가(준수/비준수 식별)
  + 이니셔티브(여러 정책 정의를 큰 수준으로 그룹화) 정의, 할당
  + 정책 정의: 사용자 정의 정책 추가 가능
  + 예) 저장소 크기는 허용된 크기로만 만들어야 함 - 위반시 리소스 생성 거부
* RBAC: 리소스의 세분화된 액세스 관리 제어
  + (AWS IAM과 유사), 역할을 만들고 사용자에게 할당
* 잠금: 개별 리소스, 리소스 그룹이 실수로 삭제되지 않도록 설정 가능(읽기 전용(수정안됨), 삭제방지 2가지)
* Azure Advisor(관리자 보안 지원): 권장사항 안내
* Azure Blueprints: 베타버전, 리소스를 다시 만들고 정책을 즉시 적용할 수 있는 재사용 가능한 환경 정의(설계도), 블루프린트 수정 시 리소스도 같이 수정되는 연동 기능 제공
* 구독관리
  + 요금 청구: 구독별 보고서 및 지불 거절
  + 액세스 제어: 구독에서 RBAC설정 가능
  + 구독 한도
* Azure 모니터: 리소스 추가 즉시 데이터 수집, 모든 리소스 생성/수정 이벤트 기록(활동 로그)
  + Azure 모니터 에이전트
* Azure 서비스 상태
  + Azure 상태: status.azure.com
  + 서비스 상태: 서비스, 지역별로 필터 후 확인 가능
  + 리소스 상태
* 응용 프로그램 및 서비스 모니터링: Azure 모니터 + Azure 서비스 -> 데이터 모니터링
* 규정 준수 약관 및 요구사항
  + 가장 포괄적인 규정(인증 및 증명 포함) 준수함
  + CJIS, HIPAA, CSA STAR, ISO/EIC 등등..
* 개인 정보 보호 - 어떤 데이터를 어떻게, 어떤 목적으로 처리하는지 공개
* 가격 책정 및 지원
  + Azure 구독: 리소스 프로비저닝을 하려면 무조건 구독이 있어야 함, 구독이 없어도 포털 로그인은 가능
  + Azure AD: 전용 Tenant제공(tenant는 독립적인 공간), 구독을 이용해서 tenant아래에 리소스를 채워 넣을 수 있음
    - 구독에 따라 경계 - 청구 경계, 액세스 제어 경계(관리 정책에 따라 구독 구분?)
    - 디렉토리 만들기로 신규 tenant추가 가능
    - 관리 그룹: 엔터프라이즈 급은 사용자가 많기 때문에 관리 그룹으로 구분, 각 그룹별 별도의 구독 사용 가능
  + 비용에 영향을 주는 요인: 리소스 유형, 서비스, 위치
  + 청구 목적의 영역: 대역폭에 대한 과금(아웃바운드의 영역 기반으로 책정)도 유의, 영역은 4개 구분, 서구선진국/나머지/브라질남부/독일
  + Azure 계산기: https://azure.microsoft.com/ko-kr/pricing/calculator, 사전 비용 예상, 총 소유 비용(TCO) 계산기 활용
  + 비용 절감: 비용 분석, Azuer Advisor, 지출한도 설정, 예약 리소스, 위치/지역 선택(고객의 위치도 고려), tag를 통해 비용 소유자 식별
* 지원 계획
  + 무료: 청구및 구독 관련, 제품 및 서비스 설명서, 도움말, 커뮤니티
  + 개발자: 평가판, 비프로덕션 환경
  + 표준
  + 프로페셔널: Azure에 의존도가 높은 조직
  + 프리미어: MS제품에 대한 의존도가 높은 큰 조직
  + 대체지원: MSDN Azure, StackOverflow, ServerFault, Azure Knowledge Center
  + SLA: 99.9%에서 99.99% 약속, 서비스 2개의 SLA는 각 SLA의 곱
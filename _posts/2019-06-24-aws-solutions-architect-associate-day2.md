---
layout: post
title:  "AWS Solutions Architect 교육과정 2일차"
date:   2019-06-24 22:00:00 +0900
categories: AWS
comments: true
---
# AWS Solutions Architect 교육과정 2일차 정리
본 글을 2015년 한국 AWS에서 교육받은 내용을 메모해 뒀던 글을 다시 정리하여 올린 글입니다.  
2015년 교육 자료이므로 현 시점(2019년)과는 차이가 있을 수 있습니다.  
용어, 한글표기 등은 시간 관계상 맞추지 않았습니다.  

## 스토리지
* EBS(Block), S3(Object)
  + 1byte를 변경하면 EBS는 16kb가 변경되고, S3는 전체가 변경됨
* EBS
  + 가상 네트워크에 연결된 블록 스토리지
  + EBS는 영구적이며 EC2인스턴스에 연결됨
  + 인스턴스가 재시작될때 EBS는 유지되고 인스턴스만 치환됨 (확인 필요)
  + EC2 1개는 여러 EBS
  + EBS 1개는 여러 EC2
  + 생성되면 동일 AZ내에서 아무 인스턴스와 연결가능 (그러나 AZ밖은 안됨)
  + 해당 AZ내에서 가용성을 위해 중복생성됨
  + IOPS
    - 범용 : 최대 3000IOPS : 스토리지 용량 * 3 IOPS제공
    - 크래딧이 적립됨(1TB 이하 까지만), burst로 사용가능
    - Provisioning : 용량 뿐아니라 IOPS에도 가격이 매겨짐 (동일 성능이지만 3배정도 비싸짐, 꼭 필요한 설정으로 사용할 것, IOPS를 높이고 싶으면 용량을 늘이면 됨)
    - 마그네틱 : 용량과 IO수로 가격이 측정되지만 느리고 싸다
  + 스냅샷
    - EBS를 특정 시점으로 백업하는 것
    - 첫 스냅샷은 전체, 이후로는 변화된 부분만 저장
    - 암호화 됨
  + 대용량(16TB)이상으로 쓰려면?
    - RAID 
      - RAID0은 속도를 높이자, striping
      - RAID1은 안정성을 높이자
      - RAID10을 쓰기도 함
      - 그외의 RAID는 비권장
  + 권장 설계 : EC2 - EBS(OS) - EBS(Data)
  + 범용 IOPS는 100%성능 보장이 아님(최대치 정도로 보면 됨), 성능을 고정적으로 보장 받기위해서는 PIOPS를 써야됨
  + 상위 EC2에서는 EBS 최적화 기능이 제공되기도 함
  + EBS스냅샷은 20기가 변경마다 또는 1개월에 1번 스냅샷 권장 (스냅샷은 S3에 저장됨. 보이진 않음)
    - 동작하는 중에 스냅샷을 찍으면 문제가 될 수 있음(컨시스턴스 문제 등, stop하고 스냅샷하는 것을 권장)
  + 루트 볼륨은 머신을 끄지 않는한 용량 변경 불가
    - 데이터 볼륨으로 별도 EBS를 사용하길 권장(기존볼륨을 스냅샷하고 큰 볼륨으로 복제 후 갈아끼우기)
  + EBS는 재사용 직전에 초기화 됨 : 즉 누군가 다시 쓰려고 할때 삭제됨
* S3
  + 고가용성 99.999999999%내구성, 99.99%가용성
  + 최종 일관성(결국엔 일관적이 됨)
    - 일관성 / 가용성 / 파티션내성 3개중 2개를 만족하면 1개는 부정됨 -> S3(타사 스토리지 또한)에는 일관성을 최종일관성으로 처리
    - 쓰기 후 읽기 일관성, 몇 초 이후엔 일관성이 보장되지만 1초도 컴퓨터 세상에선 긴 시간
      - 일관성이 보장될 때 까지 새 파일이 안보임, 삭제한 파일이 보임, 수정한 파일이 반영안됨 등이 있을 수 있음
  + 리전 내에 많은 복제가 존재
  + 버킷 : 최상위 개념, 전세계에서 고유해야함
  + 폴더개념이 없으나 보기 좋게 prefix를 폴더처럼 이용
  + 객체는 버킷+객체명이 고유해야 함
  + 업로드시 병렬화 또는 멀티파트를 쓰면 좋음(100메가 이상일때 성능이 차이남, 5메가 이상이면 적용가능)
  + 버킷 접근제어
    - IAM으로 적용가능 : 사람에게 접근권한을 줄떄
    - ACL : 개별 파일 접근 권한 줄때
    - 버킷정책 : 보통 이걸 씀, 버킷에서 접근하는 것을 통제
      - IAM policy와 비슷하나 Principal이라는 IAM 대상을 지정해 주는 부분이 존재
      - 버킷에서 이벤트(생성,삭제,깨짐) 발생시 SNS SQS Lambda로 알림을 받을 수 있음
      - 스토리지의 클래스를 바꾸면(Reduced Redundancy : 내구도가 낮아짐) 깨지는 경우가 생김
        - Standard Infrequent: 내구도는 standard 와 동일하고 저장비용이 저렴하지만, 접근비용이 큼, 로그(고용량이지만 자주안봄)같은거 저장용
        - Reduced Redundancy는 원본이 존재하고 임시 파일일 경우에 쓰면 좋음, 복제가 2개만 존재하므로 깨질 가능성 증가
        - 라이프 사이클을 통해 Glacier로 보내면 좋음 (Glacier는 4시간 후에 조회 가능함)
    - 스토리지 게이트웨이를 통해 S3를 사내 온프레미스와 연결해서 백업용으로 사용가능
    - Third Party를 통해 S3에 백업하면 편함 (S3가격은 동일)
  + VPC에서 연결시 IGW를 통과해야하지만, 이 부분을 보완한 VPC S3 Gateway가 존재하여 내부에서 연결해서 사용가능
* Amazon은 고객의 데이터에 접근하지 못함
* 리전 간 데이터 백업은 고객 스스로 해야 함

## 확장성
* ELB: 관리형 로드 밸런싱 서비스
  + 직접 만들경우 커스터마이징이 가능하지만 관리 어려움
  + ELB를 쓰면 저렴하고 간단함
  + 지원요소 : Health Check, 
    - 교차영역 로드밸런싱
      - ELB가 AZ 밖에 있는것 같지만 실제 구현은 AZ내에 구현됨 -> AZ너머로 분배도 할 수 있음
      - AZ안에서만 분배하면 응답속도가 높지만, AZ너머까지 분배하면 균등해지는건 좋음
    - 자체적으로 auto scaling하므로 자꾸 IP가 바뀌고 최소 2개존재하므로 IP로 연결하지말고 URL로 호출해야함
    - 세션 유지(스티키 세션)
      - 세션이 등록되면 세션 등록된 인스턴스로만 보내짐 (스티키세션을 사용해야 하는 경우도 있지만 유지안하는게 균등성에는 더 좋음)
    - 드레이닝 : 인스턴스 응답이 느려지면 서서히 분배를 줄임
  + 다중ELB패턴
    - 한 인스턴스 셋(오토스케일링)으로 여러 ELB를 붙임(조건에 따라, 인증서/모바일 등등)
  + ELB에서 인증서 체크도 해줌 : 이를 통해 인스턴스 별 SSL 설정을 하지 않아도 됨, 인스턴스가 1개라도 이를 위해 ELB를 쓸 수도 있음
  + ELB는 리전간 분배가 되지 않음 (Route53으로 비슷하게 구현가능)
* Auto Scaling
  + 자원을 예측하지 말고, 필요할때마다 증가시키자
  + 수평적 확장(갯수), (수직적 확장(성능)이 불가능하진 않지만 자동으로는 안됨)
  + ELB + CloudWatch + AutoScaling : 클라우드워치가 EC2의 CPU 사용량 등을 감시 -> AS정책이 실행됨 -> ELB가 증가/감소된 인스턴스를 중계
  + 최소 0, 최대의 최소값 1, 원하는값 지정 가능
  + 스레싱 : 너무 쉬운 기준으로 증가하고 줄이면 돈낭비 발생
    - 증가 : 서비스 가용성에 중요
    - 감소 : 비용 절감에 중요 (쿨다운 피리어드 설정)
  + DIY로 조건 설정 가능 : 복잡하게 할 수 있으나 보통의 경우는 자동으로 하면 됨
  + 인스턴스의 소결합을 위해 상태 비저장을 하면 좋음 - 상태 정보를 테이터 스토어로 푸시해두면 좋음 (스티키 세션을 쓰면 오토스케일링에 별로 좋지 않음, 증가/감소할때 세션이 유지되어야 하므로)
  + 필요시 병렬처리용으로 사용 가능 : 같은 비용(1시간단위)으로 속도가 빠름
  + 오토스케일링 그룹은 런치컨피규레이션을 최초 갯수만큼 런치 인스턴스 하고, ELB가 CloudWatch로 healthcheck를 하면서 정해진 기준(Scale policy)에 따라 런치 컨피규레이션을 스케일 아웃하거나 스케일 인한다.
    - 이때 ELB와 런치 컨피규레이션간에는 보통적인 EC2처럼 SG를 설정해두어야 함

## 데이터 관리
* DynamoDB : Amazon의 NoSQL, 보통 아키텍처에서 Session관리용 DB로 서버에서 분리하여 사용됨
  + EC2 필요없는 풀서비스(완전관리형)
  + 지연시간 짧고 확장 원활, 성능 예측 유리, 내구성/가용성 좋음
  + 파티션(해쉬)키 : Primary(유니크)키 같은거, 검색용
  + 복합 기본키 : 여러 키를 엮어서 유니크해짐
  + 범위(정렬)키 : 범위 검색 가능
  + 항목 : Row, 속성 : Column이지만 스키마가 없음, 400kb이하!
  + 가격이 비싸므로 최소한의 데이터를 관리하도록 하자
  + SSD에서 동작하고 확장이 자유로움(축소는 하루에 4번)
  + RCU : 1RCU = 4kb/s 
  + WCU: 1WCU = 1kb/s (쓸데는 여러곳에 써야하므로 3군데 속도가 느림)
  + 일관성을 강제해서 읽어올 수 있음, 최종일관성 정책으로 읽어올 경우 좀 더 빠름 8kb/s
  + 행/열별 액세스 제어 가능(상세한 제어 가능)
  + 데이터를 일/주/월 별로 쪼개서 관리하고 일/주/월별 접속 속도 제한을 조절하여 비용 줄이고 성능을 높일 수 있음
  + 핫키와 핫파티션을 사용하지 않음 : 내생각: 핫키(파티션)은 집중된 데이터를 위한 솔루션을 의미하는듯
    - RCU를 정할 수 있는데(많을수록 비쌈) 이 RCU를 파티션(키)이 나눠서 쓰게되는데 한 파티션에 한종류의 데이터가 몰리면 해당 파티션의 RCU로 속도가 떨어짐, 아키텍처를 변경해야하고 임시방편으로 RCU총량을 늘이면 됨(비쌈)
  + Global서비스는 아니지만 약간의 추가 작업으로 Global처럼 쓸 수 있음
    - DynamoDB를 리전별로 구축하여 event발생시 데이터를 모아서 하나의 DB처럼...
* RDS
  + 사용자가 EC2에 올려써도 되지만 아마존에서는 완전관리형 서비스를 제공
  + MySQL, Oracle, MSSQL, 자체 Aurora도 있음(성능이 빠르다네)
  + 온프레미스 > EC2 > Amazon RDS
    - EC2일 경우 튜닝이나 세부 커스터마이즈가 잘될 수 있음
    - RDS로 가면 간편하고 쉬움(패치나 가용성, 백업 관리 다 해줌), 쿼리만 잘 하면 됨
      - 대규모 읽기/쓰기나 커다란 샤딩일 경우, 간단한 NoSQL은 사용할 필요 없음
      - Oracle같은 비싼 DBMS는 고급 설정을 쓰고 싶으니까 굳이 RDS를 쓰지 않음
  + Aurora : 멀티AZ지원, 복제도 가능. 비용대비 성능이 4배정도 향상될 수 있으나 heavy할때는 느려지는 점도 있음
* ElasticCache
  + Redis나 Memcached와 호환가능하고, 유사한 기능 지원, 역시 완전관리형
  + DynamoDB 앞에 ElasticCache를 붙이면 비용절감 및 속도 향상 효과
  + 게임 업체에서 대규모 유저정보 및 빠르고 많은 상태 변경을 위해 많이 사용.

## 네트워크
* Route53 : 퍼블릭 인터넷 연결 서비스, 외부에서 DNS 라우팅
  + 도메인 등록(신규 등록도 지원해줌)
  + 헬스 체크
  + 심플 라우팅 (도메인을 IP로 바꿔주는 기본기능)
  + 가중치 기반 라운드 로빈 (웨이티드 라운드 로빈) : 재해복구 또는 업데이트 때 사용(점진적으로 트레픽을 돌릴때)
  + 지연시간 기반 라우팅 : 레이턴시가 짧은 쪽으로 보냄, 멀티 리전일때 쓰면 좋음
  + 기본 서버가 죽으면 S3의 파일을 넘겨줄수 있음(죄송합니다 복구중입니다 같은..)
  + 지원 레코드
    - A Record : Name을 IP로 바꾼 (DNS 표준)
    - C Name : Name을 Name으로 바꾼 (DNS 표준)
    - Alias(Amazon특화) : Name을 AWS endpoint로(S3, Elastic Beanstalk, ELB, CloudFront) 공짜!
      - Zone apex (풀 도메인에서 www 없는 이름)때문에라도 써야됨 : 표준상 A record만 가능함, 그러나 Alias에서 지원가능함
* CloudFront
  + S3의 정적 데이터를 빠르게 제공 : S3는 데이터 가용성 내구성 좋음 (Tip파일 이름이 동일하면 한쪽으로 몰려서 속도가 느려지므로 파일명 앞에 랜덤을 넣으면 속도가 좋음, 단 정렬이 안되서 불편하기도)
  + 대용량 파일을 고객에게 제공하려면 Torrent형식으로 제공하면 데이터 트래픽 비용 50%정도 절감됨
  + 동적 데이터도 당겨서 뿌릴 수 있음, 즉 ELB를 CloudFront를 붙여도 좋다.
  + CloudFront 앞에 WAF(웹 방화벽)을 붙여서 웹 공격을 막을 수 있다.
  + 캐시 컨트롤 헤더로 정/동적 콘텐츠 식별
  + 파일에 캐시컨트롤로 만료기간 설정 가능 (TTL변경, 객체 이름 변경)
  + 객체 무효화 : 마지막 수단, 비효율적이고 비용이 많이 듦 이라곤 하지만 쓸만한 기능(건당 비용임) AWS입장에서는 별로 지만 언제든지 써도 됨
  + 캐쉬이므로 오리진의 로드가 줄어듬
  + 사용자정의 SSL제공
  + 프라이빗한 콘텐츠로 (특정 URL로 일시적으로) 제공 가능 : SignedURL
  + OAI(원본액세스ID)를 통해 클라우드프론트에서만 S3를 접근가능하게도 할수 있음(S3에서는 접근 불가) *** 자격시험 단골문제

## 관리
* CloudWatch
  + 모니터링 : EC2 등에서 제공되는 정보를 모아서 볼수 있음
  + 또는 로그를 수집해서 필터링 해서 볼 수도 있고
  + CloudWatch정보를 Third Party로 보내서 볼 수도 있음(ELK 등등..)
  + 켜야 됨
  + 하이퍼바이저 내부의 정보는 모니터링이 안됨(메모리, 프로세스 갯수 등)
* CloudTrail
  + 사용자가 호출한 API를 조회할 수 있음
  + 다른 계정도 통합해서 조회할 수 있음
  + 켜야됨(S3비용이 쌓임, 로그가 쌓여서)

## 정리
* 다이나모디비가 엘라스틱케쉬보다 사용자가 불규칙적일때 유리 (엘라스틱캐쉬는 고정 사용자가 많을 수록 유리)
* DB는 마스터 + 스탠바이(슬레이브) 를 AZ로 분리하고 저장은 마스터로, 스탠바이로 백업. 그리고 Read Replica로 읽기 설정
* NAT인스턴스는 여러 서브넷에 나눠 놓는다고 다 쓸 수 있는게 아님(왜냐? 라우팅을 한 곳으로 밖에 못함). 그래서 NAT는 고 가용성을 위해 오토스케일링 함
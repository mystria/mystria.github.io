---
layout: post
title:  "MongoDB로 RDB 대체하기"
date:   2020-08-27 16:00:00 +0900
categories: MongoDB
comments: true
---

# MongoDB로 RDB 대체하기
본 글은 2020년 08월 MongoDB Webinar를 듣고 정리한 글입니다.  
들으면서 급하게 메모한 거라 부정확한 부분이 있을 수도 있습니다.  
MongoDB 측 세미나이기 때문에 MongoDB의 관점이기는 하지만, MongoDB가 기존 RDB를 대체할 수 있는지, 어떤점이 나은지를 엿볼 수 있습니다.  

## DB 시장 트랜드
  + On-prem -> Cloud
  + RDBMS vs NoSQL-> 내 requirement, pain point를 해결해준다면 적합[^Kakao_Bank]
  + 한 가지냐? vs Multi model 이냐? -> 어떻게 한 가지로 다양한 요구를 해결할 수 있나? -> MongoDB 한 가지로 해결 가능?
  + RDBMS에도 다양한 data type model의 add-on 지원
  + Data As A Service(DAAS)

## 현실성 체크
* 설계 다이어그램은 되게 이상적, 안정적:  
그러나 실제 구현된 아키텍처를 분석해보면 복잡하게 얽혀있고 자원 낭비가 큼
  + Data duplication and movement
  + Code complexity
  + Operation & maintenance

* 시장에서 많은 기존 대기업들이 대체될(사라질) 것으로 전망: 혁신 필요
  + 개발자에 대한 액세스[^Developer_Access]가 가장 큰 제약
  + Legacy system 이 혁신의 장벽
  + 기존 시스템 유지에 비용이 너무 큼
      - 다운타임의 1/3이 유지보수
      - 사이버 범죄, 정보 규제 등 대응 비용

* Legacy Modernization
  + 아키텍트 관점
    - 디자인(Win -> Web -> Mobile)
    - 패러다임(CICS, SOA, MSA...)
    - 기본개념(Fundamental)에 대한 도전(Make vs Buy, Upgrade vs Replace vs Nothing to do)
  + 플랫폼 관점
    - 핵심역량에 대한 재평가(나의 주 역량이 데이터 센터 운영인가?)
    - Service Level Expectation의 재설정(예전처럼 낮게 잡을 필요가 있을까?)
    - 비용 vs 기회
  + RDBMS : ORM[^ORM]을 통한 비즈니스와 DB의 매핑, Driver 학습필요, 중간에 레이어가 많다
  + MongoDB : 비즈니스 객체를 transform 없이 그대로 저장, Idiomatic Driver, 빠르고 유연, 한 종류의 DB로 다 맞출 수 있음

## MongoDB로 옮기기
1. ORM을 걷어내기, DB 엔진 교체
  + MongoDB는 공식 ORM 제공 하지 않음: 실질적으로 RDBMS용이므로 필요가 없음
1. Table vs Document
  + Document는 super set, Table도 표현 가능, ACID(Transaction)지원, 1to1, 1toN, NtoN 표현도 가능
    - 1to1, 1toN, NtoN 에 대해 embeded하든가 reference하든가
    - Document라고 Table보다 느리지 않음 (단순하게 그냥 저장만하면 느릴 수 있음)
  + 그러나 요구에 맞게 개발/액세스 패턴에 맞게 document 설계 - 스키마 설계하는 순서가 뒤집힘
  + 설계 방향: 둘다 혼용 하는게 좋다
    - Bottom up: Table to Document
    - Top down: Business logic to Table
  + [Document 디자인 패턴](https://www.youtube.com/watch?v=sycYpKOGTMA)
    - Attribute pattern: 유사한 필드가 많을 경우, 많은 필드를 한번에 검색
    - Subset pattern: Working set이 큰 경우
    - Bucket pattern: 빠르고 많은 데이터 처리(IoT 등)
    - Computed pattern: 데이터 계산이나 조작에 비용이 많이 드는 경우
1. 데이터 컨버젼: 도구제공, 다양한 practical한 절차를 통해 마이그레이션
  + 기존 데이터에 특정 타입을 변경할 때 $convert같은 aggregation으로 inplace에서 처리 가능 (새로 재적재 할 필요 없음)
  + Aggregation pipeline을 통해서 어지간히 복잡한 쿼리도 다 지원
1. 최적화
  + MongoDB도 인덱스가 있는데 활용이 떨어짐
    - compound, unique, geospatial, hash, array, TTL, Sparse/partial, text speach
  + MVIEW, Union(4.4 이후 추가) 이용
1. QA
1. Going Live: best practice play book 제공, 기술 지원도 가능

* 사례
  + Rakuten: Oracle TimesTen -> MongoDB
  + FLO: 사이즈는 반으로 줄어듬 (KV로 인해 더 늘어날 것 같지만, 스키마 재설계로 오히려 줄어듬), 3배 빨라짐, 로드밸런싱, 플렉서블
  + UnderArmour: 기존 CMS(Consumer Management System?)를 개선
  + CISCO, SEGA, Baidu: RDBMS(Oracle, MySQL, MSSQL)에서 넘어옴

## 질의
- 몽고DB도 대량 업데이트 시 bulk update를 권장함(라운드 트립 줄임)
- (기존 뉴스에 나왔던) 보안 취약점 : RoleBAC을 지원하지 않으나 일반적인 보안 설정을 하면 전혀 문제 없음
- Secondary node 에서 스냅샷 가능
- 기본적으로 데이터 압축해서 저장, collection 별로 지정 가능
- MongoDB는 Time-series데이터 처리에 적합 bucket으로..

## 개인적인 총평
현재 MongoDB Atlas를 사용하는 입장에서 많은 부분이 납득이 갔다. 특히 유지보수 비용을 무시할 수 없는데, 별도로 DBMS를 구축/관리/모니터링 을 하려면 온전히 2명 이상의 인력이 필요하다. 물론 DB가 시스템의 코어 요소라면 자체적으로 관리하는 것이 유리할 수 있겠지만, DB가 비즈니스 요구사항을 충족하기 위한 system의 일부일 뿐이라면 DaaS 등을 활용하여 빠르게 비즈니스 로직에 집중하는 것이 낫다.  
NoSQL을 쓰면 개발할 때 data type에 얽매이지 않는 것도 좋은 요소이다. 개인적으로 performance 에서도 불리함을 느끼지 못했으나, time-critical 시장에서는 어떨지 모르겠다.  
MongoDB에서 제품을 영업하는 부분은 감안해야 겠지만, 개인적인 생각으로 일반적인 서비스에서는 RDB를 굳이 고집할 필요가 없을 것 같다.

[^Kakao_Bank]: 카카오뱅크는 뱅킹 서비스임에도 불구하고 MySQL로 구축했다. 기사: [카카오뱅크는 어떻게 MySQL로 데이터 유실을 막았을까](https://byline.network/2017/10/17-6/)
[^Developer_Access]: 정확하게 무슨 뜻인지 못들었는데, 개발자가 접근하기 편하냐 또는 개발자를 구하기 편하냐 정도로 추측
[^ORM]: DB의 의존성 없이 개발 가능, 그러나 복잡한 query는 불가능
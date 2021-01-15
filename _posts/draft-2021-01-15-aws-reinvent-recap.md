---
layout: post
title:  "AWS Re:Invent 2020 요약 Webinar 정리 - 1부"
date:   2021-01-15 10:00:00 +0900
categories: AWS
comments: true
---
# AWS Re:Invent 2020 요약
AWS  Webinar를 들으며 요약했습니다. 급하게 적느라 빠지거나 틀린 부분이 있을 수도 있습니다.  
[AWS의 SlideShare](https://www.slideshare.net/awskorea)에 자료가 공유되어 있으니 자세한 내용은 직접 확인 가능합니다.  
https://www.youtube.com/awskorea
https://www.slideshare.net/awskorea

## 혁신가
* 5가지 클라우드 기술변화
  + 극강의 비용대비 성능, 지속가능성
    - ARM기반 Graviton 2 (AWS가 개발한 전용 칩) 등장, Nitro/Inferentia/Trainium 등도..
    - x86 대비 40% 나은 가격/성능
    - c6gn(graviton, nitro) 신규 인스턴스
    - 스토리지: gp2 -> gp3, GB당 20% 비용 절감 3000IOPS, io2 : 64000MAX IOPS(Provison IOPS)
    - 저전력: 엔터프라이즈 데이터센터가 탄소배출이 높음, AWS는 탄소 절감
    - EC2 Mac 출시: Mac용 SW의 테스트용 장치(M1칩 지원)
  + 기계학습의 경계 허물다
    - ML역량 확대, 누구나 모든 사람이 기계학습과 인공지능을 활용
    - GPU, CPU, 인프라 지원
    - 프레임워크, 완전관리형 서비스(SageMaker: 통합 ML 개발 역량 제공, 급격한 성장 중)
      - Data Wrangler(데이터 준비과정 시각화), Feature Store(저장, 공유), Pipelines(MLOps), Clarify(편향성 검증), JumpStart, ML Job Profile in Debugger
    - ML기능을 DB에 확대 적용: Aurora ML(관계형DB에서 직접 ML), Athena ML(데이터 레이크), Glue DataBrew(데이터 ETL작업), Neptune ML(그래프 DB), Redshift ML(데이터웨어하우스), Quciksight Q(BI, 자연어로 질문하면 답변 제공)
  + 버티컬 산업 분야로 ML 확대
    - 비즈니스 요구 부응
    - Personalize(개인화/추천 리테일), Forecast(매출/수요예측- 물류/유통), Fraud Detector(온라인 사기탐지 -전자상거래), Kendra(엔터프라이즈 검색), CodeGuru(소프트웨어 개발), Comprehend(자연어 분석 -의학)
    - 지그재그, 브랜디(Personalize), 인터파크(Forecast), 메쉬코리아
    - 제조 및 헬스케어 분야
      - Lookout for Equipment(비정상적 기계동작 탐지-엔터프라이즈 급 기업), Monitron(End-to-End 상태모니터링 시스템, 센서-중소기업), Lookout for Vision(비전으로 불량 탐지, 30개 정도의 이미지(사진)로 학습 - 결함 탐지 가능), Panorama(엣지 환경 비전 애플리케이션, 기존 CCTV/카메라 활용 PPE감지, 번호판감지), Lookout for Metrics, DevOps Guru(IT) , HealthLake(헬스케어 분야 ML)
  + 하이브리드 재정의
    - 엣지영역으로 확대
    - 기존: 클라우드와 데이터센터(On-prem)
    - 지금: 엣지환경(5G, 대도시, 거친환경(공장/유전/농장), On-prem시설(병원/가게))과 연계
    - 각 영역들 모두에게 데이터, IoT, ML 서비스를 제공 - Greengrass, SageMaker Neo, Wavelength, Snowball, Outposts, RoboMaker
    - Outposts(AWS서비스를 Lack으로 제공, 저전력 환경용 소형 lack도 제공)
    - ECS, EKS Anywhere: container 서비스를 고객환경에 설치
  + 개방형 클라우드 환경 강화
    - 오픈소스에 공헌(관리형 서비스 제공을 위한 공헌)
    - 오픈소스 재단 및 커뮤니티에 활동(Linux, Apache, 컨퍼런스 스폰서)
    - 직접운영: Bottlerocket(컨테이너 호스팅), Firecracker(microVM생성/관리), FreeRTOS, Amplify(모바일/웹 프레임워크), Open Distro for ElasticSearch, Apache MXNet
      - new : Babelfish for PostgreSQL(레거시 SQL을 Postgre로 호환), EKS Distro(EKS기반 K8s배포판), IoT Greengrass 2.0, Distro for Open Telemetry(기업용 데이터 측정), Service for Grafana/Prometheus(시각화/모니터링 완전관리형 서비스 제공)
## 개발자
## 아키텍트
## 데브옵스
## 데이터 분석가
## 데이터 과학자
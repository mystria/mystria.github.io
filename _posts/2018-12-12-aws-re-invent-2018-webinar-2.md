---
layout: post
title:  "AWS Re:Invent 2018 요약 Webinar 정리 - 2부"
date:   2018-12-12 16:00:00 +0900
categories: AWS
comments: true
---
# AWS Re:Invent 2018 요약 (2부)
AWS Webinar를 들으며 요약했습니다. 급하게 적느라 빠지거나 틀린 부분이 있을 수도 있습니다.  
[AWS의 SlideShare](https://www.slideshare.net/awskorea)에 자료가 공유되어 있으니 자세한 내용은 직접 확인 가능합니다.  
분량이 많아 2시간 세션을 1시간 씩 나눠 작성하였습니다. 두번째 세션은 잘 모르는데다 관심이 없는 주제라 많이 축약했습니다.  

## 인공지능 업데이트
  * 인프라 및 프레임워크 제공 : P3dn 타입 추가
    + Keras, GLUON
    + TensorFlow, mxnet, PYTORCH (TensorFlow는 주로 아마존에서 수행 - TensorFlow에 최적화하여 Multi-GPU에서 90%효율(기존 65%), 학습속도 2배)
  * 학습 보다 추론(Inference) 비용이 대부분(80% 정도?)
    + Amazon Elastic Inference : 최대 75%비용 절감, EC2 인스턴스에 GPU가속 기능 추가
    + Inferentia 칩셋 개발 : AWS에서 설계한 추론 맞춤형 칩셋(EC2, SageMaker, Inference 지원)
  * 어려운 ML 구축 - SageMaker 기계 학습 모델 생성, 훈련, 서비스 배포 관리형 서비스
    + Jupyter Notebook ? : Ground Truth로 신뢰도 높은 학습 데이터(라벨링 된) 세트 구축 자동화
    + 다양한 Algorithm 추가
    + ML을 위한 마켓 플레이스 제공
  * Deep Learning Compiler : SageMaker Neo를 오픈소스로 공개
  * 최근 강화 학습(잘하면 보상, 못하면 적은 패널티)의 대두
    + 작년 DeepLens, 올해 DeepRacer(자동 주행 자동차) 출시
  * AI 서비스 - Rekognition(사진 영상 인식), Lex, 등등
    + 개인화 및 추천 - 아마존 쇼핑몰의 노하우 
      - Amazon Personalize : 개인화 및 추천 서비스, EMR에 데이터 넣으면 실시간으로 API 제공
    + 비즈니스 예측 - 역시 아마존 쇼핑몰의 노하우
      - Forecast : 시계열 예측 서비스(SAP, Oracle SC, Timestream과 통합), 데이터 셋 입력 시 예측 API 제공
    + OCR 
      - Amazon Textract : OCR ++ 서비스, 텍스트 뿐 아니라 테이블 등 에서 데이터도 추출
  * AI + 분석 + 스토리지 통합
    + AI Stack을 연계하여 요구사항 충족
    + Lake Formation : Data lake 구성 관리를 쉽게도와주며 며칠 내에 안전한 data lake 구축

## 기타
  * 보안 - 최우선 순위
    + AWS Security Hub : 클라우드 환경 전반을 중앙에서 보안 관리
    + AWS Control Tower : 다중 계정, 외부 접근에 대한 접근 창구
  * 하이브리드 환경 구축 옵션
    + On-Premise 환경과 통합
    + VMware 기반으로 AWS서비스 운영 : Amazon RDS on VMware
    + Edge(네트워크가 어려운 지역)에서 데이터 이동을 위한 Snowball : EC2 레벨의 기능 지원 추가
    + AWS Outpost : On-Premise에 하드웨어를 설치하여 AWS를 local에 구축 가능
  * Well-Architected
    + 클라우드를 적절하게 잘 설계했는지 AWS 아키텍트가 점검해주며, 교육, 백서 발행
      - 운영 탁월성, 보안, 신뢰성, 성능 향상, 비용 최적화
      - 기술 비즈니스 팀에 요청하면 검사 해줌
    + Well-Architected Tool : 직접 측정 가능

## 차세대 산업(IoT, 로봇, 우주산업) 진입 장벽 제거
  * IoT : FreeRTOS, Grenngrass, AWS IoT 시리즈 + Iot SiteWise, IoT Things Graph, IoT Events 추가
  * 로봇 : RoboMaker(오픈소스 운영체제, 개발환경, 시뮬레이션 환경 지원)
  * 우주산업 : Ground Station(저가, 소형 위성 Cubeset을 관리해 줄 수 있는 서비스)

AWS가 다양한 기능을 제공해 주기 때문에 나중에 뭔가를 만들 때 AWS에서 기능을 찾아볼 수 있었으면 좋겠다.  
Go Build!  


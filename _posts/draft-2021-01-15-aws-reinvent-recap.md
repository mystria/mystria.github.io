---
layout: post
title: "AWS Re:Invent 2020 Recap 요약 Webinar 정리"
date: 2021-01-15 10:00:00 +0900
categories: AWS
comments: true
---
# AWS Re:Invent 2020 Recap 요약
AWS Re:Invent 2020의 Recap Webinar를 들으며 요약했습니다. 급하게 적느라 빠지거나 틀린 부분이 있을 수도 있습니다.  
[AWS의 YouTube](https://www.youtube.com/awskorea), [AWS의 SlideShare](https://www.slideshare.net/awskorea)에 자료가 공유되어 있으니 자세한 내용은 직접 확인 가능합니다.  

## 혁신가
* 5가지 클라우드 기술변화
  + 극강의 비용대비 성능, 지속가능성
    - ARM기반 Graviton 2 (AWS가 개발한 전용 칩) 등장, Nitro/Inferentia/Trainium 등도..
    - x86 대비 40% 나은 가격/성능
    - c6gn(graviton, nitro) 신규 인스턴스
    - 스토리지: gp2 -> gp3, GB당 20% 비용 절감 3000IOPS, io2: 64000MAX IOPS(Provison IOPS)
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
      - new: Babelfish for PostgreSQL(레거시 SQL을 Postgre로 호환), EKS Distro(EKS기반 K8s배포판), IoT Greengrass 2.0, Distro for Open Telemetry(기업용 데이터 측정), Service for Grafana/Prometheus(시각화/모니터링 완전관리형 서비스 제공)

## 개발자
* 서버리스
  + (향후 보강)
    - Lambda 1ms 별 과금 (기존 100ms)
      - Microtask에도 Lambda를 적용 - 보통 batch에 사용 중인데, cache가 완벽하게 걸린 GET요청의 경우(10~20%) 5~10ms인데, 이걸 billing이 줄어 이제 실제에도 활용 가능

* Serverless Database 새로운 기능들
  + 전체 시스템을 serverless로 해야 serverless를 쓰기 좋음(왜냐하면 scaling이 다 같이 되어야 안정적)
  + Aurora 업데이트
    - Aurora Serverless (MySQL, PostgreSQL 완벽호환) 처음부터 Cloud기반 설계 - 1/10 가격, 3~5배 빠른 성능, read replica 충분, 지속적 백업 - 데이터유실이나 downtime없음, DB버전 업그레이드 자동
    - v2에서 Scalability 강화됨(v1의 제약들(Multi-AZ, Global DB, ReadReplica 미지원)이 거의 해결됨) - througput이 적을때 더 싸짐, Throughput에 변동에 대한 반응이 더 좋아짐
    - 아직 GCP나 MS엔 완전한 Serverless DB없음
  + DynamoDB 업데이트
    - S3로 Export가능 (Table단위로 json으로 바로 S3로 빼줌)
    - API Gateway - Lambda - DynamoDB 이런 구성이 기본적, 이때 대용량 분석을 하려면 DB에서 처리하기엔 비효율, 동일 데이터를 Kinesis를 통해 S3에 저장하여 Athena에서 분석하는 수를 썼음(Sync문제, 데이터 중복, 관리 까다로움), 이제 DynamoDB를 S3로 주기적으로 dump뜨고 이 데이터를 분석하면 간편

* 그외 새로운 서비스들
  - Workflows for Apache Airflow를 관리형 서비스로 제공: AirBnB에서 만든 big data분석용 도구(Data pipeline을 DAG로 구성, 이를 처리), ETL, AI/ML데이터 전처리, DevOps의 log분석
  - Service for Grafana를 관리형 서비스로 제공: 데이터 비주얼라이제이션, 보통 대시보드

* 모든 시스템을 Serverless로 처리 가능하나?
  + Catch Fashion: 명품 제품을 여러 리테일러에서 판매 - 가격 비교/재고 확인, 많은 데이터(불규칙한 수많은 제품 이미지)를 수집/실시간 처리
    - 요구사항: 리테일에서 제공하는 제품의 이미지들을 제품 샷과 착용 샷으로 구분
    - 기술적으로 불규칙하고 대량의 정보가 발생 - on-demand로 처리하고자 함 - serverless로 처리하기로
    - 메모리(3GB)에서 ML추론, API Gateway로 data입력
    - tensorflow.js를 수행, pre-trained model사용
    - image URL - Lambda에서 받아 memory에 담아 buffer - tensorflow에서 받아 추론 후 분류
    - 나아가 pre-trained model을 이용해 착용샷 사람의 관절을 분석, 앞뒤도 구분 - 이런식으로 기능을 추가하여 제품의 위치까지 찾음
    - 이미지를 넣으면 다양한 메타 데이터를 추출, 이를 활용하여 적절하게 고객에게 노출
    - Lambda가 cold start때 S3에서 model을 다운로드 받음 - 200MB이지만 몇 초 만에 다운로드 가능, 간단한 모델이라면 가능
    - 커스텀 모델은 S3에 배포하는 pipeline구축, 트레이닝 후 S3에 업데이트, Lambda가 이를 활용 - Lambda 재시작 시 최신모델 사용
    - tensorflow 성능 문제 - tensorflow.js는 좀 느림, C++바인딩 활용 EFS 사용해야 함 - 비싸지만 20배 빠름, WebAssembly구현체(Browser에서 동작, but NodeJs에도 동작) 5배 빠름(EFS불필요)
* Audience Question
  - Q: 컨테이너로 사용하게 됐을 때 콜드스타트는 기존과 차이가 없나요?
    - A: 네 없다고 합니다. 그래서 굳이 10GB 이야기를 다시 언급한건데, AWS Lambda 팀이 내부적으로 10GB 기준으로 돌려도 런타임 콜드스타트가 기존의 native function (300MB 이하의) 가 별차이 없도록 성능개선 작업을 많이 했다고 들었습니다.
  - Q: 저희 사내에서 AWS 클라우드 인프라를 구축하여 서비스를 운영중입니다. ECS(jar 호스팅) 및 EC2(DB, prometheus, grafana)를 사용하여 운영중입니다. 개발 CI/CD 또한 EC2에서 jenkins, sonarqube, sentry를 호스트방식으로 구축하여 사용중인데 이를 서버리스 환경으로 재구성하기 위해 어떤방법을 사용하여야 좋을지 조언 부탁드리겠습니다.
    - A: 마이그레이션 계획과 순서를 정하시는게 가장 중요하실 것 같은데요,
      1. 코드를 변경하지 않아도 되는 인프라들을 옮긴다. (DB, CI/CD등...) 기존에 사용하고 계신 솔루션들의 SaaS 대응을 찾아서 옮기시면 될것 같습니다. (github actions / aurora rds 등)
      2. application 코드를 옮긴다. 이것도 점진적으로 쉬운 기능들부터 쪼개서 옮기는게 좋을텐데, 좀오래된 내용이긴 합니다만 원칙은 그대로라서요, 아래 영상 참고하시면 좋을 것 같습니다.
        - https://www.youtube.com/watch?v=CM47-1UpgOc
        - https://www.youtube.com/watch?v=-rvZ4Ft5qQE
  - Q: Lambda로 구현하는 대표적인 비지니스 프로세스에는 어떤 것들이 있을까요?
    - A: 제가 일했던 회사들 (캐치패션 / 빙글) 모든 프로세스를 전부 Lambda로 구현했습니다. "모든" 에는 HTTP API, Web Frontend, WebSocket, Cron, ETL, Queue worker등이 다 포함됩니다.
  - Q: aurora for postgresql과 RDS for Postgresql 성능차이는 어떤 서비스가 더 좋은가요? 
    - A: aurora for postgresql은 개념적으로 DB engine 자체는 aurora 이고, 쿼리 인터페이스만 postgresql과 호환도되록 지원하는 개념입니다. 다시 말해 성능은 aurora의 성능이고요, 아마 발표 전반부에 잠깐 언급했던 것 같은데, 같은 하드웨어 스펙일때 3배정도 성능 차이가 난다고 벤치마크 되어있습니다.
  - Q: aws 오랜만에 봐서 처음 알았는데요, aurora가 서버리스를 제공하면 굳이 서버리스가 아닌 버전(용어를 모르겠네요)를 쓸 이유가 있을까요? 가장 큰 차이는 무엇이 있을까요?
    - A: 말그대로 "서버리스인지 아닌지가" 다릅니다. aurora serverless는 앞에서 언급한것처럼 하드웨어 스케일링에 대해 개발자가 아예 손을 안되도 되는걸 가정하는 구조입니다.
  - Q: DB에 쌓인 데이터를 S3 로 Export를 하여 저장한 후 차후에 S3에 저장한 DB 데이터를 복원할 수도 있나요? 할수 있다면 관련자료 사이트 참조부분할 수 있는데도 있을까요?
    - A: S3 -> DynamoDB를 native로는 지원하지 않는걸로 알고 있고요, 대신 DynamoDB 를 back / restore 하는 기능은 존재합니다. 만약 반드시 S3 -> DynamoDB를 하셔야 하면, 직접 insert를 만드셔야할듯.
  - Q: AMG(Aws Managed for Grafana), AMP(Aws Managed for Prometheus)를 사용하게 되면 기존 EC2 호스트형식으로 구성한 것과 차이점을 알수 있을까요?
    - A: 다른 AWS Managed 서비스들 과 마찬가지로 아래와 같은 이점이 있습니다.
      - 보안 관리
      - 소프트웨어 버젼 업데이트
      - 기본 플러그인들 관리
      - 하드웨어 관리를 안해도 됨
  - Q: aurora serverless의 경우 rds proxy에서 지원하지 않아 connection limit 오류가 발생할 여지가 있어서 Lambda와 같이 사용하는 것이 망설여져서 아직까지 rds+rds proxy+Lambda 구조로 사용하고있는데, aurora serverless를 어떻게 활용하는지 알수있을까요?
    - A: 아무래도 스트레스 테스트 (이 경우엔 당연히 new connection 스트레스 테스트 일 것 같은데) 를 해보셔야 할 것 같긴 한데요,  
    원론적으로 RDS Proxy가 하는건 이 connection을 공유하는 버퍼를 만들어주는거라서요. Aurora Serverless는 master-master 구조로 스케일링을 제공하니 만약 스케일링이 Lambda가 스케일링 되는거랑 같은 속도로 일어난다면 new connection은 문제가 없어야 하긴 합니다.  
    발표에서도 언급했지만 aurora serverless v1은 스케일링이 그래도 몇분씩 걸려서 이 문제가 완벽하게 해결되진 않았는데, v2에서는 벤치마크가 필요한것 같네요.  
  - Q: eks/fargate로 container orchestration 하는 것에 비해 Lambda container image가 비용적으로/성능적으로 더 나을 수 있다고 보시나요?
    - A: 케바케일 것 같습니다. 지금 발표에서 언급하는대로 traffic fluctuation이 많거나 cold start가 엄청나게 길지 않은 작업들이라면 Lambda가 비용/성능 둘다 이득이 있을꺼고, 그 반대이거나 CPU / GPU / Memory를 더 뽑아써야 한다면 EC2기반 솔루션들도 고려해보셔야 할 듯 합니다.
  - Q: 전체 시스템을 서버리스로 구현하는 것의 장점은 무엇인가요? 언뜻 생각하기에는 AWS 서비스들도 대부분 운영에 필요한 리소스가 온프렘에 비해 대폭 감소해서, 마이그레이션을 한다면 먼저 ec2 등등을 활용하는 게 더 간편할 것 같은데 전체 서비스를 서버리스로 구현하기로 하신 이유가 궁금합니다.
    - A: 마이그레이션을 한다면 당연히 순차적, 부분적으로 하는게 맞다고 생각합니다. https://www.youtube.com/watch?v=CM47-1UpgOc 이걸 참조하시면 될 것같고, 이득은 아래와 같습니다.
      1. 개발자들이 DevOps 관련된 일을 전혀 하지 않아도 됩니다.
      2. 개발자들이 스케일링을 고려하면서 코딩할 필요가 없어집니다.
      3. 실제 서비스를 운영할때 트래픽을 보면서 스케일링을 수동으로 하거나, 서비스 상태 모니터링을 사람이 해야하는 요소가 극단적으로 줄어듭니다.
  - Q: ECS Fargate 도 Saving plan 을 이용하여 요금절감을 할 수 있는 것으로 아는데 어떻게 적용할 수 있는건가요? Saving Plan 인터페이스에서 설정하면 되는것인가요?
    - A: EC2 서비스 들어가는것처럼 서비스 항목에서 Saving plan 에 들어가시면 Compute Saving Plan 을 구매하시면 됩니다.  
    "입력하는 금액이 $1 을 입력하더라도, 1년 or 3년치이기 때문에 금액은 우리 월금액에 맞춰 직접 계산을 해보아야합니다."라고 컨테이너 히어로이신 주영님께서 말씀하시네요.
  - Q: Lambda로 구현한 로직이 많아질 경우 이런 로직을 관리할 수 있는 솔루션이 필요하지 않을까요?
    - A: 여러 Lambda function 목록을 관리하는거라면 딱 Lambda에 특화된건 아니지만 service catalog tool이라고 해서 마이크로서비스 목록을 관리해주는 오픈소스 툴은 여러가지가 있습니다. 제가 일하는 곳에서는 커스텀하게 Lambda 목록을 만들어주는 소프트웨어를 직접 만들어서 쓰고 있습니다.
  - Q: 서버리스에 적합한 비지니스 환경은 어떤것들이 있을까요?
    - A: 아직까지 서버리스로 해결 못할만한 비즈니스 도메인을 별로 보지 못해서... 머신러닝 트레이닝이나 비디오 인코딩 정도?
 

## 아키텍트
* 솔루션즈 아키텍트: AWS이용하는 고객에게 Cloud전략 소개, 전환 업무 지원, 아키텍처 설계 및 구축, PoC및 프로토타이핑, 이슈해결지원
* EC2 인스턴스: 광범위하고 심층적인 컴퓨팅 플랫폼 지원
  - General Purpose, Bustable, Compute intensive, Momory/Graphic intensive, Storage, Dense storage, GPU compute, 
  - Processor, Memory, ...
  - EBS, Elastic Inference
  - 위의 조합으로 수 백 가지 서비스 제공
* 신규 인스턴스
  + Apple 개발자 용
    - 현재 Xcode이용, 온프레미스(개인 하드웨어), Mac-as-a-Service(제한된 글로벌 생태계), CI-as-a-Service(멀티테넌트의 runner, 커스터마이즈 어려움)
    - EC2에 macOS지원 시작: 애플 개발자용 구축, 인프라는 신경 쓸필요 없음, 민첩성 탄력성 보안 지원, AWS와 긴밀한 통합
      - 빌드 팜 확장으로 빌드 시간 절감
    - Mac인스턴스: 6 cores, 12 logical core, 32GB, 10gbps / 애플 하드웨어 , 니트로 보안칩, 니트로 카드?
    - AWS와 통합(IAM, VPC, CloudWatch...), 빠른 글로벌화, 온디멘드 요금제
    - mac1.metal(베어메탈 타입), macOS 및 Xcode 네이티브 지원, AMI로 카탈리나/모하비 제공(빅서는 2021 상반기)
    - 단일 테넌트 "전용 호스트"로 사용가능, 최소 24시간 단위 이용, 호스트가 부족할 경우 다른 AZ에서 요청 필요
    - SSH, VNC로 접근 가능 (현재 US East, US West, EU아일랜드, AP(싱가폴))
    - sudo passwd ec2-user (permission) 필요
  + 새로운 타입
    - D3/D3en 출시: 대부분의 클라우드 스토리지 보다 훨씬 더 큰 빅데이터 워크로드, 336TB, 45%/100%높은 디스크 처리량
      - D3: 로컬 HDD(6~48TB), 3.1GHz, 3.1MiB/, 25Gbps
      - D3en: 로컬 HDD(28~336TB), 3.1GHz, 6.2MiB/, 75Gbps - 다중노드 파일시스템, 대용량 워크로드
    - C6gn 출시: C5대비 40% 가성비, Elastic Fabric Adapter(EFA)지원, 100Gbps, 고성능 컴퓨팅/네트워크 어플라이언스/실시간 비디오 통신
    - G4ad 출시: G4dn대비 45%나은 가성비, AMD Radeon 구동, 게임 스트리밍, 그래픽 렌더링
    - M5zn: M5보다 45%가성비, Xeon확장형 프로세서,  CPU별 라이선스가 필요할 때 고성능으로, 시뮬레이션/분석/게임 워크로드,EFA지원
    - R5b: 최대 7500MB/s 대역폭, 260000IOPS EBS, CPU별 라이선스가 필요할 때 고성능으로, 관계형 DB 및 분석/디스크처리량 높은 워크로드
    - Trainium: 기계학습용 TFLOPS 컴퓨팅 성능, Tensorflow, MXNet, PyTorch와 같은 일반적인 ML지원, SageMaker 통해서 이용가능, 이미지인식/음성인식/자연어처리/등 ML에 효과적
    - Habana Gaudi기반 인스턴스: 현재 GPU대비 40% 가성비, 고객이 모델을 더 자주 반족 학습가능

* 스토리지
  - EBS io2 Block Express: 까다로운 워크로드 용 최초 SAN 스토리지, io2보닫 4배더 높은처리량, 1ms초급 지연, 데이터가 크고 I/O집약적인 요구사항에 적합, 4GiB-64TiB
  - gp3: 데이터베이스, 컨테이너, 빅데이터, 파일시스템, 미디어에 적합, 3000IOPS(gp2 1000배), 1000MiB/s(gp2 4배), 20% 낮은 가격
  - gp2는 용량 당 IOPS설정됨, gp3는 적은 용량으로도 높은 IOPS할당 받음 (추가비용내면 별도 설정 가능)

* Aurora Serverless v2
  - Aurora: 클라우드용 MySQL, PostgreSQL. 상용제품보다 1/10비용, 높은 처리량, 여러 데이터 센터에서 읽기/쓰기, 15 replica, 3개AZ에 6개복제, S3에 연속적백업, 지역간 복제가 가능한 단일 글로벌 DB
  - 서버리스에 대한 요구 증가, 사용자의 유지보수 불필요
  - 용량관리 불필요, 수십만개 트랜젝션으로 즉시 확장 가능, 부하에 대한 프로비저닝 비용 90%절감, 높은 가용성(다중 AZ)
  - 고성능 어플리케이션 지원, 서버리스에 최적화(빠른확장, 빠른 입력처리)

* 데이터베이스에 대한 AWS의 접근방식
  - 관리형 서비스
  - 마이크로 서비스, 특수목적에 적합한 서비스 제공
  - 마이그레이션 지원


## 데브옵스
* 데브옵스 기본적 정의와 이해: 문화, 자동화, 측정, 공유, 축적 - 개념도 계속 진화
  - 주요업무: 서비스 개발과 배포를 신속 정확, 서비스와 팀의 문제를 빠르고 정확하게 감지, 장애를 최대한 빨리 해결, 확장가능/신뢰할 수 있는 아키텍처 설계, 데이터에 기반한 원활한의사소통 지원 - 이를 위한 도구 적용/전파
* (안정적이고 신뢰할만한)AWS 인프라가 중요 - AWS 덕에 하드웨어적으로는 쉬워졌지만 아키텍처적으로 새로운 과제
  - 도전적인 문제에 대해 다양한 해결방법, 모범사례가 존재할 수 있음
  - 인프라관리 방법 - 중앙관리 | 자체 솔루션 | PaaS
    - 중앙관리: 어느정도 되는 규모의 조직이어야함, 표준화 용이, 티켓관리 등 필요, 개발자들 전체적으로 수준이 높아야 함
    - 자체 솔루션: 적합한 플랫폼 구축 용이, 일부 인원에 의존성이 생김
    - PaaS: 쉽고 빨리 적용, 커스터마이징이 어렵고, 플랫폼에 락인 됨, 진화/다양한 요구 수용 어려움
* Proton 출시
  - AWS가 최초로 제공하는 컨테이너와 서버리스를 위한 완전관리형 배포 서비스
  - 인프라엔지니어가 의도한 표준과 모범사례를 준수할 수 있음, Proton을 사용하지 않더라도 많은 모범 사례를 참조할 수 있음
  - 인프라 관리자 -(인프라스트럭쳐 template을 정의/생성)-> Proton <-(내 앱에 적합한 템플릿을 선택하여 배포)- 개발자
  - IaaS기반, CICD파이프라인, 가시관측성 모니터링 툴 필수
  - 3가지 핵심기능: Self-service 인프라 개발자를 위한 코드기반 배포, 중앙관리의 이점(표준화) 원클릭 업그레이드, 외부 솔류션과의 통합
  - 환경(Environment): 실제 서비스가 위치하게될 VPC와 cluster같은 기본 공유자원 정의(모범사례 존재)
  - 서비스: 서비스 자체의 구성
  - 환경템플릿 구성(인프라개발자, 관련지식이 있는 개발자), 서비스 템플릿 구성
  - Web console, CLI, SDK제공
  - 모든 환경을 (Code화 하여)일치시켜, 가격 효율성 신뢰성 구축
  - 특정값의 파라미터화 강제화 - 동일한 템플릿으로 다양한 환경의 실제 서비스 생성 -> 인프라를 코드화
  - 템플릿 관리(설명, 버전관리-메이저/마이너), 샘플 템플릿 제공(Well-Architected, Open source(GitHub))
  - 개발자는 템플릿을 골라서 적절한 파라메터를 입력하여 모범사례의 인프라를 구축하고 배포할 수 있음, 배포된 서비스를 간략한 대시보드로 확인 가능
  - 아직 퍼블릭 프리뷰 (클라우드 포메이션 기반 템플릿 작성, 템플릿 기반 서비스와 환경 배포, AWS Codepipeline과 연계)
  - 향후 헬스 모니터링, 중요 외부서비스와 통합, 테라폼 등, 자동태깅, 멀티 어카운트, 템플릿 접근제어 준비 중
  - 목표: 고객에게 컨테이너와 서버리스 배포 모범사례를 적용(IaaC사상 전파), 컨테이너와 서버리스 손쉽게 구축, 비즈니스 성공을 위한 아키텍처에 집중(구축을 간단히, 인프라팀 부담절감)
* 사람과 일이 늘어날 수록 느려지는 개발 속도를 모범 아키텍처와 인프라 자동화로 다시 높이기 위한 노력 필요
* Audience Question
  - Q: Proton이 ElasticBeanstalk나 LightSail의 상위호환같이 좀 더 다양한 템플릿으로 제공하게 된 것 같은데요. 이들과 차이점이 있을까요?
    - A: 조금 더 선택지가 많습니다. 인프라를 정의하고 설정하고 결국에 SG 설정부터  VPC 설정까지 모든걸 포함하는게 Proton 입니다. 훨씬 더 범위가 큽니다. 그리고 자유도도 높습니다~ ElasticBeanstalk 에 내부 배포 방식을 변경하는게 가능하다고 보면 될 것 같네요.
  - Q: 인프라스트럭처 템플릿을 정의하고 생성한다고 하셨는데, CloudFormation도 비슷한 역할을 하는 것을 알고 있습니다. Proton과 CloudFormation의 차이점은 무엇인가요?
    - A: Proton이 CloudFormation + @ 라고 생각하면 일단 이해하기는 편합니다. Terraform, CloudFormation 도 어떻게 어떻게 하다보면 구축부터 배포까지 모두 할 수 있지만, 그 과정이 단계화되어있지 않고, Status and Health 모니터링이 되는건 아닙니다. Proton 은 Application 배포, 즉 Lambda 와 컨테이너 배포가 포함되어있습니다. 유사한 서비스를 언급해본다면, serverlessframework 와 닮아있다고 보시면 됩니다.


## 데이터 분석가
* 데이터 분석이 어려운이유
  - ETL? 데이터를 준비하는 과정: Extraction & loading > Cleaning & normalization > Orchestrating at scale
  - 데이터 준비에만 80% 시간 소요 - Data engineer, ETL developer -> Data analyst, Data scientist
  - 데이터 엔지니어는 데이터 수집하는 서버도 구축해야함, 가공하기 위해서는 파이프라인 구축도 필요
  - Data analyst를 하려고 하는데, 그 전에 데이터 준비하는데 노력이 많이 필요함
    - Python이나 Scala같은 언어로 데이터 변환하는 코드 작성
    - 코드로 작업
    - VPC내 S3로 데이터 옮겨야함
    - 위와 같이 시간이 많이 소요되는 작업의 반복
* Glue DataBrew
  - 데이터 분석가와 과학자를 위한 도구
  - 데이터 품질에 대한 이해, clean & normalize data, 데이터 상관관계의 맵 시각화, scale의 자동화
  - 데이터 전처리
* Demo
  - Project 생성(Recipe 지정)
  - 샘플링 사이즈 설정, 퍼미션 설정(기본생성)
  - profile 생성/실행
  - Job 생성 - 출력을 저장할 S3지정, timeout/retry설정, 퍼미션 설정(기본생성, S3 putObject필요)
  - Recipe(데이터 가공 절차-불필요한 컬럼 삭제, 컬럼 이름/타입 변경, 필터링, 그룹핑) 빌드 - publish


## 데이터 과학자
* SageMaker (2019년 출시)
  - CICD 템플릿으로 ML 워크플로우 구성 및 관리
  - 수천개의 모델 버전별 관리
* SageMaker Studio에서 사용 가능
* SageMaker Pipelines
  - CICD를 ML에 적용
  - ML은 반복(고통): 데이터 준비 -> 모델학습 -> 모델배포 -> 데이터 준비 -> 반복
  - 배포하더라도 운영에 적용할 때 반복 -> 본인이 원하는 결과를 얻을 때 까지
  - ML 워크플로우 구축의 어려운 점
    - 단계별 처리과정이 많음, 단계별 디펜던시 관리
    - 단계별 산출물 관리, 만들어진 모델의 버전 관리
    - Scalable한 자동화 환경 구축
  - 워크플로우 구축은 Python SDK제공
  - 파이프라인 상태 시각화 실시간 제공: 단계별 상태, 결과/로그 제공, 단계별 파라미터 변경 가능
  - 모델 레지스트리: 모델 관리(ECR과 비슷) - 메트릭을 메타데이터로 함께 버전으로 관리됨, 계정도 관리가능
  - 비개발 과학자들도 쉽게 template으로 pipeline 구축 가능

* Data Wrangler
  - 데이터 준비작업(전처리)을 쉽게 도와주는 관리형 서비스
  - 데이터 선택, 클리닝, 피쳐 엔지니어링, 시각화, 모델 프로토 타이핑 등...
  - 다양한 도구를 사용하기 위한 환경 구축 - 노력이 많이 소요, 
  - 재작업/실수 가능성, 다른 환경에서 재작업(transition)필요
  - 코드작업 없이 전처리/이상값제거/형변환 가능
  - 다양한 데이터 소스(Athena, Redshift, Lake Formation, S3, Feature Store등)
  - 손쉬운 데이터 변환(Missing, Outlier, ...)
  - 손쉬운 데이터 시각화(Histogram, Scatter Plot, Box Plot..)
  - 모델 성능에 대한 빠른 예측: 데이터 준비과정 중 이상점, 피처간 상관관계, 의사결정 사항을 준비단계에서 확인할 수 있음
  - 손쉬운 프로덕션 배포: Export 또는 Pipeline으로 통합, Feature store에 publish

* SageMaker Feature Store
  - Feature는 모델의 입력값(ML의 입력값) - RawData를 코드화하여 Feature vector
  - Feature Engineering: 복잡한 연산, 길고 지루한 작업, 반복
    - 결과 재사용/공유 어려움
    - Feature 사용을 위해 재작업 필요하기도 함(ML환경/Production환경이 달라서) - 변환과정에서 오차 가능
  - 제공 방식
    - Streaming Ingestion: 스트리밍 처리위한 API제공
    - Batch Ingestion: Custom Spark Container 제공
  - Online / Offline Store
    - Online: 스트리밍, 최신Feature, ms단위 데이터 동기
    - Offline: Historical 데이터(parquet), 15분 데이터 동기

* SageMaker Clarify
  - Bias Detection 필요: 편향성을 찾아야함 - 모델이나 데이터 자체의 편향성을 이해 필수, 서비스 차원이 아니라 법적 문제까지 가능
  - DeepLearning의 경우 Modeler도 자신의 Model을 설명하기 힘듦 - 설명가능성과 성능은 trade off, 다양한 설명가능성을 위한 테크닉 존재
  - Bias detection이나 설명가능성을 위한 API제공, pipeline에 integration
    - 데이터 전처리 단계 / 모델 학습 이후: Bias Reporting
    - 모델 학습 이후: Model Explanation, Bias Drift
    - 모델 배포 이후: Explainability Drift


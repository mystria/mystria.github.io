---
layout: post
title:  "AWS DevOps 교육과정 1일차"
date:   2018-11-09 09:00:00 +0900
categories: AWS DevOps
comments: true
---
# AWS DevOps 교육과정 1일차 정리
본 글을 2018년 11월 한국 AWS에서 교육받은 내용을 메모해 뒀던 글을 다시 정리하여 올린 글입니다.  
용어, 한글표기 등은 시간 관계상 맞추지 않았습니다.  

## DevOps의 개요
  - 개발 효율화, 반복작업 배제 : 보안, 안정성
  - 개발과 운영 조직의 고립된 환경(silo) 극복 : 개발 배포 프로세스의 매끄러운 관리(convergence)
  - 지속적인 통합(CI) / 지속적인 전달(CD)
  - 지속적인 통합 : 매일 매일 빌드(코드 리포지토리, 빈번한 커밋) - 빌드 자동화
  - 지속적인 전달 : 배포 - 반복적인 작업을 코드로 관리(=코드형 인프라)
  - SecDevOps : 보안, 감사, 검증 - 커밋 또는 배포 시 보안적으로 문제가 없는지 등을 체크

## AWS CLI
  - AWS의 api를 cli로 실행, python기반 (web management console의 세부 기능을 수행)
    ~~~ 
    $ aws <option> <service> <operation> <parameters>
    ~~~
  - output : json, text, table
    ~~~
    $ aws configure로 credentials, region등을 설정
    ~~~
  - 설정 파일은 로컬에 credentails, config 파일로 저장됨
  - —profile 을 통해 프로파일(= 개별 권한 설정)을 생성하여, 향후 명령 수행시 프로파일 지정하여 실행 가능
  - —query 옵션 : JMESPath를 이용해 필요한 값만 추출해서 조회가능(jmespath.org)
  - waiter : cli작업이 시간이 걸릴 경우 waiter 를 사용하여(내부적으로 15초 간격으로 polling) 상태가 변화면 다음 단계로 넘어 갈 수 있음
  - —generate-cli-skeleton : 작업 내용을 json형식으로 파일저장, 나중에 이걸 읽어서 동일한 작업 수행하게 할 수 있음
  - 인증서 찾기
    - 우선 순위 : export AWS_ACCESS_KEY_ID=… > credentials파일 > config파일 > AssumeRole > 인스턴스의 메타데이터
  - AssumeRole을 이용하여 role관리

  - 권한을 account간에도 부여 가능 -> account너머로 받은 role을 다시 내부 IAM단위로 부여, 부여받은 사람은 account 너머의 역할을 수행가능
    - ex) AccountManager라는 account가 별도로 있어서 모든 account들로 부터 IAM관리 role을 위임받고, 다른 account의 사용자관리를 대신해 줄 수도 있음

## 코드형 인프라(Infrastructure as Code)
  - 목적
    - 환경을 너머 동일한 인프라 구축
    - 빈번하며 즉각적인 테스트
    - 설정/구성의 지연 제거
    - 자체 문서화
    - 이점 : 인프라의 버전 관리, 반복적이고 안정적인 인프라 재생성, 최신 애플리케이션을 테스트하기 쉬움
    - 커스텀 스크립트 vs CloudFormation(인프라구축) vs OpsWorks(어플리케이션 레이어)
* AWS CloudFormation
  - AWS자원의 배포를 위한 선언적 스크립트 언어
  - AWS인프라 관리를 단순화, 인프라 복제가능, 인프라의 문서화/제어/추적가능
  - Bootstrapping
    - EC2 자원 설치 시 OS단에서 이뤄져야할 작업은 user data 스크립트로 구성해야 함
    - user data 의 실행은 사용자 스크립트(cloud-init)와 Cloud Formation 초기화(cfn-init) 진행
    - cloud-init : 
    - cfn-init : cloud formation의 스크립트로 OS단 작업 실행(구문화 되어 있음) - cloud formation이 진행사항 체크 가능함, 단, cloud-init에서 cfn-init을 호출 해줘야 함
  - 커스텀 자원 : cloud formation이 지원하지 않는 자원, third party 자원 포함 가능
  - 스택 업데이트
    - cli로 간단히 업데이트 가능 : 단 어떤건 중지없이 업데이트 가능하지만, 어떤건 중지 후 업데이트, 대체 될 수 있음
    - cloud formation을 업데이트 하지 않고 개별적으로 업데이트 한다면 일관성 유지, 버전관리, 문서화가 안됨
  - 스택 삭제 : 삭제 후 취소 불가, 모든 자원이 삭제되지만 일부를 백업하려면 Deletion Policy설정(retain)을 하면 됨, 수동(cloud formation 외적)으로 백업해둔 것도 남음
  - 스택 설계 시 계층적 아키텍처로 접근
  - 리소스 생성 뿐 아니라 계층별 구조로 관리
  - 스택 운영
    - 스택 체이닝 - 스택간의 연계(a 스택의 output을 b 스택의 parameter로..)
    - IAM으로 스택 관리는 cloud formation으로만 하기 위해서 : 리소스 이름이나 태그로 삭제/운영기능 제한
    - S3에 스택 탬플릿을 담아 공유도 가능

* 구성관리 : 인스턴스 구성을 스크립트 화
  - Chef(Ruby), Puppet(Python), Ansible
  - 멱등적(1회만 실행되어야), 강제적(template외의 변경은 무시되어야함), 분산화(여러 인스턴스에 전달되어야)
  - cloud formation 의 cfn-init에서 인스턴스에 chef등을 설치 할 수 있도록 설정 가능

## IAM
  - Management Console은 ID/PW로 접근하지만 내부적으로는 AccessKey/SecretKey로 동작. 이외에는 모두 AK/SK로 동작됨
  - 권한 타입 : 사용자 기반 / 자원 기반
  - 사용자 기반 : 누가 어떤 자원에 무엇을 하는가?
  - 자원 기반 : 특정 자원에 누가 접근 가능한가?
  - Role : 사용자 또는 자원에게 부여가능
  - 역할 기반 접근 관리 : 논리적/기능적 그룹에 단기 인증, 벌크 관리(확장가능), 임시권한 제공 -> AssumeRole
  - IAM 정책 : 권한을 문서화 시켜 둠, Role/사람/자원등에 붙일 수 있음
  - Deny우선 - allow와 같이 있으면 deny 적용됨
  - 미리 정의된 정책, 정책 생성기, 정책 시뮬레이터 활용
  - 평가로직 : 암묵적 거부 > 명시적 거부 > 명시적 허용 (기본적으로 모두 제한됨)
  - 외부 인증시스템으로 자격 연동 가능(LDAP등..)
  - 연동을 지원하는 관리형 서비스 제공 : AWS Directory Service
  - STS(Security Token Service) : 제한된 접근을 제공하는 임시 인증서 제공
  - 최소 권한 원칙 : 최소한의 권한만 부여하고 역할 전환으로 업무 수행
  - MFA 지원 : api 호출 할 때 mfa인증을 활용(루트 계정이나 장기 인증서에는 미적용)

## AWS Lambda
  - Serverless. Stateless(상태유지를 위해서는 s3나 db에 저장 필요)
  - 실행권한과 실행역할? 분리

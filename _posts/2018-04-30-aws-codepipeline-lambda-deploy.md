---
layout: post
title:  "AWS CodePipeline 사용하기"
date:   2018-04-30 20:00:00 +0900
categories: AWS CodePipeline Lambda CICD
comments: true
---
# AWS CodePipeline을 이용하여 Lambda를 배포하기
  * 요구사항
    + GitHub Enterprise(이하 GHE)에 있는 AWS Lambda용 소스코드를 CICD를 적용하여 자동으로 배포
  * 문제점
    + CodePipeline의 Source 단계(stage)는 GHE를 아직 지원하지 않음
  * 해결책과 설계
    + CodeBuild는 다행히도 GHE를 지원
    + CodeBuild에서 GHE의 코드를 shallow clone하여 build하고, artifact를 S3에 업로드
    + S3에 파일이 업로드 되는 것을 trigger로 하여, CodePipeline을 시작

## AWS Code Series
  * AWS CodeBuild
    + [AWS CodeBuild란?](https://docs.aws.amazon.com/ko_kr/codebuild/latest/userguide/welcome.html)
      - 빌드 작업은 간단한 경우도 있지만, 엄청 무겁고 오래걸리는 경우도 있음
      - 이런 경우 CodeBuild를 이용하면 이용한 시간만큼만 비용 발생 - 빌드 용 클러스터 구축 비용 절감
    + GHE로 부터 코드를 가져와 빌드: [Announcing AWS CodeBuild Support for GitHub Enterprise as a Source Type and Shallow Cloning](https://aws.amazon.com/ko/blogs/devops/codebuild-support-for-github-enterprise/)
      - 빌드된 artifact는 S3에 업로드
    + CodePipeline 내부에서 Build 단계로 존재 할 수 있음
      - GHE가 아니라면(일반 GitHub, CodeCommit 등) CodePipeline의 Source 단계를 통해 소스코드를 불러와 Build 단계에서 BuildSpec에 정의된 대로 빌드
      - 빌드된 artifact는 Output Artifact로 S3에 업로드되어 다음 단계의 Input Artifact로 전달 가능

  * AWS CodePipeline
    + [AWS CodePipeline이란?](https://docs.aws.amazon.com/ko_kr/codepipeline/latest/userguide/welcome.html)
    + Source 단계에서 S3의 CloudWatch로 부터 update를 기다림
      - S3 외에 GitHub, CodeCommit도 가능
      - 그러나 GHE에서 가져와야 하므로, CodeBuild에서 S3로 artifact 업로드
    + S3 변경 시 해당 패키지 파일을 가져와 Output Artifact로 등록
    + 이미 CodeBuild에서 build 되었기 때문에 Build 단계는 건너뜀(No Build)
      - 보통은 여기서 CodeBuild를 수행하게 되어있음(Jenkins 등)
      - 이 artifact를 CodeDeploy나 CloudFormation으로 배포해야 함

  * AWS CodeDeploy
    + [AWS CodeDeploy란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/codedeploy/latest/userguide/welcome.html)
      - 사견: EC2 instance의 경우 artifact를 서버에 배포하고, 배포된 서버를 기존 서버와 치환하는 작업을 수행한다. 그러나 Lambda의 경우, 이미 수정되어 versioning이 된 함수들을 배포 모델(BlueGreen, Canary 등)에 맞춰 치환하는 작업만 수행한다.
      - Application, Deployment Group, Deployment Configurations와 Revision으로 구성됨
      - Application은 하는 역할이 없음, 그냥 Project같은 개념
      - Application 아래 Deployment Group은 Deployment Configurations중 한가지 배포 모델을 선택하고, Trigger나 Rollback, Alert을 설정하지만 마찬가지로 특별히 하는 역할은 없음
      - Revision이라고 하고 Deployment라고도 하는(=Deploy new revision)게 CodeDeploy에서 가장 큰 역할
    + Revision 생성
      - Application을 선택하여 대상 플랫폼(EC2/Lambda등)을 결정
      - Deployment Group을 선택하여 배포 모델(AllAtOnce등)과 기타 옵션을 결정
      - AppSpec을 수기입력 받거나 S3로 지정하여 실질적인 배포를 수행
      - 사실상, AppSpec이란 것이 CodeDeploy의 전부
      - 앞서 언급했듯이, AppSpec으로 (EC2에 배포할 경우에는 artifact를 지정할 수 있는 것 같음(files)) Lambda에 배포할 경우 버전 exchange만 가능함
    + 정리: CodeDeploy로 Lambda를 배포하는 것은 불가능
      - 개발자가 console이나 CLI를 이용하든 CloudFormation(이하 CF)든 Lambda 함수의 artifact(소스코드)를 수정하고, 함수 버전을 publish 해두면, Deployment Group에 지정된 배포 모델을 적용하여 Alias를 seamless하게 변경해주는 것이 전부
      - 만약, 제가 잘못 이해했다면 댓글 부탁드립니다.

## 작업 기록
CodeDeploy 상태를 봐선, 수동 또는 CF를 이용해서 Lambda코드를 수정하는 것은 CodePipeline의 역할인 것으로 보임: [관련 문서](https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/build-pipeline.html)

  * SAM(Serverless Application Model)
    + [AWS SAM 사용](https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/serverless_app.html)
    + [Spec](https://github.com/awslabs/serverless-application-model/blob/develop/versions/2016-10-31.md#awsserverlessfunction)
    + CloudFormation형식을 이용한, Serverless 앱 구축을 정의한 모델(템플릿)으로, CF에 stack으로 등록 시 "Transform"하여 일반 CloudFormation template으로 변환됨

  * CodePipeline 구축
    + 위 "관련 문서"에서 안내하는대로 작업
      - Lambda 함수는 CodePipeline의 Deploy단계에서 Deployment Provider로 CF를 선택하여 배포
      - Template은 SAM 형식으로 Input Artifact 내부에 위치: MyApp::sam.template
      - 만약, Action Mode를 "Create or update a stack"으로 하게되면 Create(Update)Stack cannot be used with templates containing Transforms. 이라는 에러가 발생
      - CF에서 stack으로 배포할 때는 그냥 Create Stack을 하면 됨
      - 단, console이 아닌 CLI로 배포시, create-stack이나 update-stack이 아닌 deploy명령으로 실행 해야 함
      - 참고: [Update cloudformation stack from aws cli with SAM transform](https://stackoverflow.com/a/41490589/8350542)
      - 헷갈리지 말고 Action Mode를 "Create or replace a change set"으로 선택 할 것
    + ChangeSet
      - SAM은 CF template 형식을 따르지만, 변환 과정을 거쳐야 진정한 CF template이 됨
      - SAM 내용을 CF 형식으로 바꾼 것이 ChangeSet
      - SAM을 ChangeSet으로 transform하고, ChangeSet을 실행(Execute)하여 stack을 update
      - "관련 문서"의 2단계 참조
      - 잊지말고 CodePipeline에 "Execute a change set"이란 step을 하나 더 만들어야 함
  * ChangeSet의 함정
    + SAM을 ChangeSet화 하고, 이를 이용해 stack을 update 할 때 주의사항
      - CF의 Update stack은 특성상 *바뀐 부분만 적용* 됨
      - SAM으로 Lambda stack을 update 할 때, CodeUri라는 property를 지정해야 함
      - CodeBuild로 S3에 업로드된 artifact의 파일명이 바뀌지 않으면, CodeUri부분이 변경되지 않기 때문에, stack이 update되지 않음
      - 즉, S3에 올려둔 artifact가 변했더라도, CF는 인식 불가(ChangeSet의 생성 결과:  FAILED - No updates are to be performed.)
    + 해결책
      - S3에 업로드 할 artifcat이름을 매번 바꿈? 불가 - artifact이름이 바뀌면, CodePipeline의 Source 단계를 trigger할 수 없음
      - CodePipeline의 Source 단계에서 다음 단계로 전달할 Output Artifact를 이용
      - Output Artifact는 다시 임의의 S3로 전달되는데, 이 bucket과 key는 매번 임의의 이름으로 생성되므로 이를 SAM parameter로 전달 - Parameter Overrides
      - CodeUri가 변경되므로 ChangeSet에서 "Modify"로 반영 됨

  * CodeUri 바꾸기
    + Parameeter overrides 적용
      - Deploy 단계에서 Deploy Action의 Advanced 항목을 펼쳐보면, parameter값을 override 할 수 있음
      - 참고 1: [AWS CodePipeline 파이프라인에서 파라미터 재정의 함수 사용](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/continuous-delivery-codepipeline-parameter-override-functions.html)
      - 참고 2: [Using Intrinsic Functions](https://github.com/awslabs/serverless-application-model/blob/master/HOWTO.md#using-intrinsic-functions)
      - 참고 3: [Not possible to have CodeUri as a parameter](https://github.com/awslabs/serverless-application-model/issues/61)
    + 실수 사례들
      - parameter overrides를 적용했는데, SAM template에서 parameter를 받을 준비(Parameters 정의)가 되어 있지 않으면 에러 발생: Parameter values specified for a template which does not require them.
      - 반대로 paramters를 받을 준비는 됐지만, parameter overrides 설정이 되지 않아도 에러 발생: Parameters: [BucketName, ObjectKey] must have values.

  * 기타: AutoPublishAlias
    + 이 property를 지정하면
      - 자동으로 version이 추가됨
      - 해당 version으로 alias를 지정함
    + SAM에서 별도로 Lambda 함수의 version을 생성하는 것은 불가능
      - [Safe Lambda deployments](https://github.com/awslabs/serverless-application-model/blob/develop/docs/safe_lambda_deployments.rst)

  * 정리
    - GHE -> CodeBuild -> CodePipeline(Source: S3 -> No Build -> Deploy: CF(Create a ChangeSet) -> Deploy: CF(Execute a ChangeSet)) -> Lambda function

## 해야할 것 들
  * CodeDeploy로 production 배포
    + 위 작업들로 Lambda 함수를 새로 생성하고 Alias까지 부여함
    + 운영에 배포해야 할 Lambda 함수는 배포 모델을 적용하는 것이 필요
      - CodeDeploy의 Application/Deployment Group을 이용하여 배포 모델 적용
      - Production에 mapping된 alias를 CodeDeploy로 치환
    + 이를 위한 AppSpec을 어디에 둘 것인가?

## Sample codes
  * 특이 사항
    + SAM의 CodeUri는 spec상 무조건 zip/jar로 압축되어 S3에 있어야 함
      - [배포 패키지 생성](https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/deployment-package-v2.html)
    + CodePipeline의 Source 단계에서 가져오는 소스파일 역시 zip으로 압축되어 있어야 함
      - Artifact가 일종의 package이기 때문
    + 그러므로, 소스파일 내부에 SAM template를 넣어두는 것도 가능
    + CodeBuild에서 artifact를 만들 때, SAM template도 같이 묶어 활용
      - SAM의 CodeUri로 자기 자신을 참조(단, 위에 설명했듯이 Parameters로 동적으로 참조할 것)
      - 이것이 Best Practice는 아니겠지만, SAM을 repository에 관리하는 것도 번거로움: 의견 주세요.
  * Artifact file structure
    + zip파일
      - sam.template
      - test.py
  * SAM template
  ~~~ yaml
    # sam.template
    AWSTemplateFormatVersion: '2010-09-09'
    Transform: AWS::Serverless-2016-10-31
    Parameters:
        BucketName:
            Type: String
        ObjectKey:
            Type: String
    Resources:
      PipelineTest:
        Type: AWS::Serverless::Function
        Properties:
          # 예제이므로 python을 사용
          FunctionName: pipeline-test
          Handler: test.lambda_handler
          Runtime: python3.6
          CodeUri:
            Bucket: !Ref BucketName
            Key: !Ref ObjectKey
          AutoPublishAlias: dev
          Tags:
            Owner: MYS
            Purpose: test
          Description: Test function
  ~~~
  * Python code
  ~~~ python
    # test.py
    # 예제이므로 python을 사용
    def lambda_handler(event, context):
      # Sample
      print("Hello World!")
      return 'Hello from Lambda'
  ~~~
  * Parameter overrides
  ~~~ json
    {
      "BucketName" : { "Fn::GetArtifactAtt" : ["MyApp", "BucketName"]},
      "ObjectKey" : { "Fn::GetArtifactAtt" : ["MyApp", "ObjectKey"]}
    }
  ~~~

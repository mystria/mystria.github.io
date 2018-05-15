---
layout: post
title:  "AWS CodeBuild의 Artifact 생성 방법차이"
date:   2018-05-15 22:00:00 +0900
categories: AWS CodeBuild S3
comments: true
---
# AWS CodeBuild의 Artifact 생성 방법의 차이
CodeBuild에서 build완료 후 산출물(Artifact)를 S3에 업로드 할 수 있다.  
그리고 업로드된 Object를 기준으로 CodePipeline을 trigger할 수 있는데, 여기 함정이 있다.  

## CodeBuild와 CodePipeline연계
  * CodeBuild에서 Artifacts
    + "Artifacts: Where to put the artifacts from this build project" 항목을 수정하면 Build 산출물을 S3에 업로드 가능
      - Type: Amazon S3
      - Name: 산출물이 저장될 장소의 이름
      - Path: Object앞의 prefix(Block Storage에는 폴더 개념이 없으나, 폴더처럼 사용됨)
      - Namespace type: Path와 Name사이에 위치할 변수(BuildID), optional
      - Bucket name: S3 버킷 이름, 기존 버킷 중에서 선택
    + 기본 설정으로는 S3://BucketName/Path/BuildID/Name 폴더에 산출물이 생성됨
    + Advanced settings에서 Artifacts packaging을 Zip으로 선택할 경우, S3://BucketName/Path/BuildID/Name.zip으로 산출물이 생성됨(Zip파일 안에 산출물들이 포함됨)
  * CodePipeline의 시작
    + CodePipeline의 Source를 Amazon S3로 지정 가능
      - Action category: Source
      - Source provider: Amazon S3
    + 이때 가져올 Source는 반드시 Zip파일이어야 함
      - Amazon S3 location: S3://BucketName/Path/BuildID/Name.zip 또는 S3://BucketName/Path/BuildID/Name/object.zip

## CodePipeline을 trigger하는 CloudWatch Rules
  * CodePipeline의 S3 change detection
    + 두 가지 방법: CloudWatch, Periodical polling
      - CloudWatch를 권장함
      - CloudWatch는 사실 CloudWatch Rules로 Event pattern을 기반으로 trigger
      - 자동으로 생성되는 Rule의 Event pattern
      ~~~ json
        {
          "source": [
            "aws.s3"
          ],
          "detail-type": [
            "AWS API Call via CloudTrail"
          ],
          "detail": {
            "eventSource": [
              "s3.amazonaws.com"
            ],
            "eventName": [
              "PutObject"
            ],
            "resources": {
              "ARN": [
                "arn:aws:s3:::BucketName/Path/Name/lambda-function.zip"
              ]
            }
          }
        }
      ~~~

## CodeBuild의 S3 PutObject의 차이와 문제점
위에서 언급했듯이 산출물을 Name폴더에 업로드하거나, Name.zip으로 업로드 하는 2가지 방법이 존재함
  * Artifact를 S3에 업로드하는 두 가지 방법
    1. 산출물이 Zip파일이므로, 그냥 Name폴더에 업로드
      - Event pattern에 match되지 않아 *동작하지 않음*
    2. 산출물이 여러개이므로, Name.zip으로 압축하여 업로드
      - Advanced settings에서 packaging 설정 필요
      - 자동으로 생성된(기본) Rule로도 잘 동작함

## 해결책
  * Name폴더에 그냥 업로드 하는 것은 사실 PutObject가 아닌, MultipartUpload
  * 다음과 같이 CloudWatch Rule의 Event pattern을 변경
    + "CompleteMultipartUpload"을 eventName에 추가
    ~~~ json
      {
        "source": [
          "aws.s3"
        ],
        "detail-type": [
          "AWS API Call via CloudTrail"
        ],
        "detail": {
          "eventSource": [
            "s3.amazonaws.com"
          ],
          "eventName": [
            "PutObject",
            "CompleteMultipartUpload"
          ],
          "resources": {
            "ARN": [
              "arn:aws:s3:::BucketName/Path/Name/lambda-function.zip"
            ]
          }
        }
      }
    ~~~

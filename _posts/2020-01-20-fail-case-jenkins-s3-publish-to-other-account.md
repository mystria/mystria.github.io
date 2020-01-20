---
layout: post
title:  "금주의 실패사례 - Jenkins job이 Gradle build시 file 생성"
date:   2020-01-20 18:00:00 +0900
categories: Jenkins S3
comments: true
---
# Jenkins에서 (다른 AWS account)의 S3로 artifact 배포
Jenkins에서 JavaScript로 된 artifact를 S3에 배포하고자 하였는데, 그 Jenkins의 AWS profile과 업로드 하려는 S3가 서로 다른 계정이라 업로드 후 권한이 없어졌고, 이로 인해 누구도 다운로드 할 수 없는 문제가 발생했다.  

## 목표
* JavaScript는 static한 요소로 서버에서 제공해도 되지만, S3에 배포 후 CloudFront를 통해 CDN으로 제공할 수도 있다. 이에 대한 장점 설명은 생략.
* 위와 같은 구성을 위해서는 조건이 필요하다.
  + S3의 파일이 직접 외부에 노출되면 안됨
    - CloudFront의 Origin access identity만 접근 허용
    - OAI 설정 방법: https://aws.amazon.com/premiumsupport/knowledge-center/cloudfront-access-to-amazon-s3/
  + Jenkins에서 S3에 업로드 
    - Jenkins에는 [S3 publisher plugin](https://wiki.jenkins.io/display/JENKINS/S3+Plugin) 설치 필요
    - Jenkins system configuration에서 Amazon S3 profiles 설정 필요
* 제약 사항
  + 본인의 Jenkins에는 pipeline을 위한 별도의 AWS계정이 연결되어있음
    - Amazon S3 profiles 에 등록되는 AWS계정 A(AAAAAAAAAAAA)
  + 배포하고자 하는 S3는 다른 계정
    - Bucket(이하 TARGET_BUCKET)이 위치한 AWS 계정 B(BBBBBBBBBBBB)

## 이슈
* 계정 A의 profile로 계정 B에 업로드하기
  + 계정 B에 CloudFront와 각종 서비스들이 설정되어 있으므로 계정B의 bucket에 업로드하기로 결정
  + 계정 B의 bucket에 permissions > bucket policy를 정의
  ~~~ json
  {
      "Sid": "Allow to upload from A",
      "Effect": "Allow",
      "Principal": {
          "AWS": [
              "arn:aws:iam::AAAAAAAAAAAA:root"
          ]
      },
      "Action": [
          "s3:PutObject"
      ],
      "Resource": [
          "arn:aws:s3:::TARGET_BUCKET",
          "arn:aws:s3:::TARGET_BUCKET/*"
      ]
  }
  ~~~
  + Jenkins에서 Post Build에 Publish artifact to S3 Bucket 설정
    - 빌드 후 artifact가 성공적으로 업로드 됨
    - 사용 방법은 다음 사이트 참고: http://blog.naver.com/chcjswoda/221069905153
  + 그러나,
    - TARGET_BUCKET은 계정 B에 위치하기 때문에,
    - 업로드 된 object의 permission에는 Access for object owner 와 Access for other AWS accounts 둘 다 비어있음
    - 계정 A가 다른 계정 B에 업로드 할 때 ACL을 설정하여, TARGET_OBJECT의 full control을 계정 B에게 이양해야 함(별도의 옵션 필요)
    - Full control을 넘기는 방법: https://aws.amazon.com/premiumsupport/knowledge-center/s3-require-object-ownership/
    ~~~ ssh
    $ aws s3api put-object-acl --bucket awsexamplebucket --key example.jpg --acl bucket-owner-full-control
    ~~~
    - 자세한 설명: https://medium.com/artificial-industry/how-to-download-files-that-others-put-in-your-aws-s3-bucket-2269e20ed041
  + Jenkins에서는 불가능
    - S3 Plugin에서 ACL 기능 미지원(한참 기능 업데이트가 없음): https://issues.jenkins-ci.org/browse/JENKINS-20851
    - 단, 본인 계정에 업로드 할 경우, 정상적으로 Access for object owner가 등록됩
    - Execute shell과 같은 작업으로 ACL 설정을 하고 싶지만, Amazon S3 profile에 접근 불가
    - 복잡하게 profile을 읽어오거나, Plugin을 고치거나...

## 해결
* S3는 복제(Replication)기능을 제공
  + Jenkins에서 계정 A의 임시 bucket으로 업로드
    - Access for object owner가 정상 등록
  + 계정 A의 bucket에서 계정 B의 TARGET_BUCKET으로 복제(management > replication) 설정
    - 이때 "Change object ownership to destination bucket owner(객체 소유자를 대상 버킷 소유자로 변경)" 옵션을 이용해 권한 이양
    - https://aws.amazon.com/ko/blogs/korea/amazon-s3-introduces-same-region-replication/
    - https://aws.amazon.com/ko/blogs/korea/new-cross-region-replication-for-amazon-s3/
  + TARGET_BUCKET에서 정상 서비스 제공
    - 완성된 bucket policy
    - Bucket policy로 많은 내용을 설명 가능
    ~~~ json
    {
      "Version": "2008-10-17",
      "Id": "1",
      "Statement": [
        {
          "Sid": "Distribute by CloudFront",
          "Effect": "Allow",
          "Principal": {
            "AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity XXXXXXXXXXXXXX"
          },
          "Action": "s3:GetObject",
          "Resource": "arn:aws:s3:::TARGET_BUCKET/*"
        },
        {
          "Sid": "Allow to replicate from A",
          "Effect": "Allow",
          "Principal": {
            "AWS": [
              "arn:aws:iam::AAAAAAAAAAAA:root"
            ]
          },
          "Action": [
            "s3:GetBucketVersioning",
            "s3:PutBucketVersioning",
            "s3:ReplicateObject",
            "s3:ReplicateDelete",
            "s3:ObjectOwnerOverrideToBucketOwner"
          ],
          "Resource": [
          "arn:aws:s3:::TARGET_BUCKET",
          "arn:aws:s3:::TARGET_BUCKET/*"
          ]
        }
      ]
    }
    ~~~ 
    - 참고로, TARGET_BUCKET이 복제를 받기 위해서는 versioning이 활성화 되어 있어야 함
    - 그리고 기존 파일은 복제되지 않고, 신규로 추가된 파일부터 복제가 적용됨

## 참조
* 파이프라인에서 s3Upload호출: https://stackoverflow.com/questions/42074736/jenkins-pipeline-how-to-upload-artifacts-with-s3-plugin

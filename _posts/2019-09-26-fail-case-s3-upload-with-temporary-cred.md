---
layout: post
title:  "금주의 AWS 실패사례 - Temporary Credentials 로 S3에 POST로 파일 올리기"
date:   2019-09-26 03:00:00 +0900
categories: AWS S3 EC2 IAM
comments: true
---
# EC2의 Instance Profile로 AWS S3 POST로 업로드

## 목표: Browser에서 S3에 file을 직접 올리자
Browser에서 직접 S3에 POST를 이용해 파일을 올리기 위해서는 AWS Signiture Version 4 를 이용할 수 있다. 
이를 사용하면, S3로 부터 pre-signed된 URL을 받을 수 있고, 특정 시간 내로 이 URL을 이용해 파일을 올릴 수 있다.  
그런데 이 pre-signed URL을 만들기 위해서는 access key가 필요한데, 보통은 IAM User의 access key를 이용한다.  
단, 이 IAM User의 access key를 관리하는 데는 여러 비용과 위험이 발생하기 때문에, 보통은 서비스가 동작하는 EC2의 instance profile(대놓고 노출 되진 않지만, IAM User처럼 access key를 구할 수 있다)을 사용하라는 게 AWS의 권장사항이자 best practice 이다.  
그러나 S3 대한 예제 및 guide는 도통 찾을 수 없었고, 이를 풀어가던 그 험난한 과정을 정리한다...  
  
처음에는 갈피를 잡지 못해 사방 팔방을 헤매어야 했고, 해결책을 찾고난 후에 다시 보니 곳곳에 이미 답이 있었다.
여기에는 내가 헤멘 순서대로가 아닌, 답을 찾아가는 과정 순으로 정리하였다.  

1. AWS Signature Version 4  
https://docs.aws.amazon.com/AmazonS3/latest/API/sigv4-authentication-HTTPPOST.html  
자세한 설명은 못하겠지만, HTTP POST를 이용해 S3에 파일을 올리기 위한 규약(format?)이다.  
이게 사실 원흉인데, IAM User의 access key를 이용하여 사용할 때는 아무 문제가 없다. Access key와 token을 생성하여 URL에 담아 browser에 전달해 주고, browser는 이 URL에 POST로 파일을 보내면 업로드가 된다.  
[사용예시](https://docs.aws.amazon.com/AmazonS3/latest/API/sigv4-post-example.html)를 참고  

2. IAM User의 access key의 제약  
보안 및 여러가지 복잡한 사정으로 IAM User의 access key를 더 이상 사용할 수 없게 되었다. 사실 아무리 secret key가 노출되지 않더라도 사용자의 access key가 public에 공개되는 것도 부담스럽고, 이를 주기적으로 바꾸는 것도 쉽지 않다.  
이런 이유들로 인해 AWS는 API를 사용할 때는 EC2 Instance Profile을 이용하도록 권장하고 있다. EC2에서 동작하는 SDK는 기본적으로 credential provider chain을 갖고 있는데, 별도로 지정하지 않는다면 instance profile이 chain에 의해 자동으로 지정된다. 이 instance profile은 일정시간(아마 1시간?) 마다 갱신되므로 안전하다. 단, 이 instance profile에 적절한 권한을 넣어주는 것을 잊으면 안된다.  
그래서 당연히 이 EC2 Instance Profile에 S3권한을 넣어주면, instance profile의 access key로도 AWS Signiture Version 4가 될 거라 믿었지...  
하지만 안. 된. 다.  
(InvalidAccessKeyId 라는 오류가 반환된다.)  

3. 문제 원인의 결정적인 힌트  
AWS의 defect인지, 아니면 S3 권한 문제인지.. 단순히 나의 실수인지 한참을 헤맸다. 여러가지 문서들을 읽던 중 EC2 Instance Profile의 access key는 temporary access key 라는 것을 알아냈다.  
https://stackoverflow.com/questions/39920505/unable-to-access-s3-file-with-iam-role-from-ec2#comment67135836_39920505  

4. Session Token  
위 댓글을 기반으로 AWS 문서를 확인해보니 [To make a call outside of the instance using temporary security credentials (for example, to test IAM policies), you must provide the access key, secret key, and the session token.](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html#instance-metadata-security-credentials) 라는 문구를 볼 수 있었다. (모르고 보니 잘 안보이더니...ㅠㅠ)  
즉, temporary security credential을 이용하려면 session token이 필요하다는 것인데, 이 부분은 뒤에 이야기 하겠다.  
그리고 AWS Signature Version 4 의 다른 문서에도 다음과 같이 session token이 필요하다는 안내가 있다.  
[If you are using temporary credentials, they expire after a specified interval, including the session token. You must update your session token when you request new credentials. For more information, see Using Temporary Security Credentials to Request Access to AWS Resources.](https://docs.aws.amazon.com/neptune/latest/userguide/iam-auth-connecting.html)  
그리고 위 두 문서에서 공통적으로 안내하는 Temporary Security Credentials 문서를 보자.  

5. SessionAWSCredential  
[Temporary Security Credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_use-resources.html) 여기서는 다음과 같이 말하는데, When you make a call using temporary security credentials, the call must include a session token, which is returned along with those temporary credentials. AWS uses the session token to validate the temporary security credentials. 즉, temporary credential을 이용할 때는 session token이 필요하다.  
이 session token이 포함된 SessionAWSCredentials은 AssumeRole을 이용해 만드는데, 어쨋든 SDK를 활용하기 위해서는 기존의 instance profile이 아닌 assume role을 통해 얻은 별도의 Session Credential이 필요하단걸 알게(필요하다고 착각하게) 된다.  
S3에 POST로 업로드 할 때(AWS Signature Version 4)는 x-amz-security-token, 즉, token이 필요하다는 것을 다음 문서에서 확인할 수 있다.  
https://docs.aws.amazon.com/AmazonS3/latest/API/RESTObjectPOST.html  

6. 중간 정리  
자, 여기까지 정리하면, EC2 Instance Profile로는 안된다. 왜냐하면 IAM User 와 달리 temporary 하기 때문이다.  
(착각이다, 처음엔 EC2의 temporary말고 새로운 temporary가 필요한 줄...)  
일단 session token이 들어있는 session credentials가 필요하다. 그래야 token을 구하고, 파일을 업로드할 수 있다.  
그러면 새로운 session credentials은 어떻게 만드는가?
DefaultCredentialProviderChain을 이용해서 권한을 얻은다음 STS(Security Token Service) client로 만들면 된다.  
처음 부터 내 질문의 답은 이 글에 있었다. 다만 내가 영어를 잘 못했을 뿐...  
[StackOverflow: Browser uploads to s3 with instance roles](https://stackoverflow.com/questions/18884683/browser-uploads-to-s3-with-instance-roles)  
그런데 읽다보니 답변에 STS는 필요없다고 되어있네?  

7. 최후의 함정 - STS를 호출 할 수 있는 권한  
EC2 Instance Profile로는 STS를 호출 할 수 없다. 오직 IAM User만 가능하다.  
https://stackoverflow.com/questions/35873012/error-when-ec2-running-as-a-role-tries-to-get-a-session-token-in-aws  
STS가 필요없다는 이유가 바로 이것 이었다.  

8. 다시 원점으로  
결국 EC2 Instance Profile이 이미 훌륭한 temporary credential이므로 단지 이 instance profile의 token을 구하면 된다.  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html#instance-metadata-security-credentials  
문서를 확인해 보면 Instance Metadata에 들어있다고 한다. 이걸 어떻게 꺼내지?  

9. EC2MetadataUtils  
SDK를 찾아보면 EC2MetadataUtils이란 것이 있고, 거기서 .getIAMSecurityCredentials()를 하면 Map형식으로 metadata를 반환 해 준다. Token은 어떤 key로 저장되어 있을까?  
https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/util/EC2MetadataUtils.html  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html  
https://stackoverflow.com/questions/41287920/how-to-get-accesskey-secretkey-using-java-aws-sdk-running-on-ec2  
Map을 찍어보면 key는 말그대로 role name이다. ARN같은게 아닌 그냥 name이다.  
EC2 Instance Profile의 이름을 알아내어 token을 꺼내면 된다. EC2 Instance Profile정보는 EC2의 metadata에 있는데, 이 부분 역시 EC2MetadataUtils.getIAMInstanceProfileInfo()으로 구할 수 있다.  

1. 결론
* AWS Signature Version 4를 이용해 S3에 파일을 올리기 위해서는 Session Credential이 필요함
  + Session Credential의 access key와 token이 필요함
* Session Credential은 User Credential이 STS를 통해 만들 수 있음
  + EC2의 Instance Profile 역시 STS를 통해 생성된 temporary session credential
* Instance Metadata에 있는 EC2 Instance Profile의 token을 획득
  + EC2 Instance Profile의 access key와 token을 전달하여 AWS Signature Version 4을 완성
먼길을 돌아왔지만 정답은 바로 앞에 있었다.  

## 참고
* AWS의 Credential provider chain
  + V1: https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/credentials.html
  + V2: https://docs.aws.amazon.com/sdk-for-java/v2/developer-guide/credentials.html

* Pre-signed URL을 얻는 방법
  + https://medium.com/@labcloud/s3-pre-signed-url-%EB%AF%B8%EB%A6%AC-%EC%84%9C%EB%AA%85%EB%90%9C-url-%EB%A7%8C%EB%93%A4%EA%B8%B0-596aff8bdc45
    - EC2 Instance Profile로 Pre-signed URL을 만드는데, 자세히 보면 URL의 Query param에 x-amz-security-token가 존재한다.
  + https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/dev/PresignedUrlUploadObjectJavaSDK.html
    - Java SDK로 pre-signed url을 만드는 방법, 이 역시 IAM User access key로 만들기 때문에 별로 도움은 안됨

* 조금 상관없었지만, 어쨋든 실제 x-amz-security-token을 사용한다는 걸 알게된 글
  + https://github.com/minio/minio-go/issues/785

* 이걸 Java SDK로 구현하는 예제
  + https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/dev/AuthUsingTempSessionTokenJava.html
  + STS로 새로운 temporary access key를 만들면 됨

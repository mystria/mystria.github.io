---
layout: post
title:  "금주의 AWS 실패사례 - Temporary Credentials 로 S3에 POST로 파일 올리기"
date:   2019-09-26 03:00:00 +0900
categories: AWS S3 EC2 IAM
comments: true
---
# ElasticBeanstalk에 SpringBoot 배포를 하자

작업일지

목표,
Browser에서 직접 S3에 POST를 이용해 파일을 올리기 위해서는 AWS Signiture Version 4 를 이용할 수 있다.
이를 사용하면, pre-signed되 url로 파일을 올릴 수 있다.
그런데 이 pre-signed url을 만들기 위해서는 access key가 필요한데,
IAM User의 access key가 아니라, EC2의 instance profile을 사용하고 싶었다.
그 험난한 과정을 정리한다...


처음에는 갈피를 잡지 못해 사방 팔방을 헤매어야 했고, 해결책을 찾고난 후에 다시 보니 곳곳에 이미 답이 있었다.
여기에는 내가 헤멘 순서대로가 아닌, 답을 찾아가는 과정 순으로 정리하였다.

1. AWS Signature Version 4
https://docs.aws.amazon.com/AmazonS3/latest/API/sigv4-authentication-HTTPPOST.html
자세한 설명은 못하겠지만, HTTP POST를 이용해 S3에 파일을 올리기 위한 규약(포멧?)이다.
이게 사실 원흉인데, IAM User의 access key를 이용하여 사용할 때는 아무 문제가 없다.
https://docs.aws.amazon.com/AmazonS3/latest/API/sigv4-post-example.html
사용 예시

2. 보안 및 여러가지 복잡한 사정으로 IAM User의 access key를 더 이상 사용할 수 없게 되었다.
EC2 Instance Profile을 이용하면 해당 EC2에서 동작하는 SDK는 기본적으로 그 EC2 Instance Profile에 맞는 권한으로 자원을 사용할 수 있다.
그래서 당연히 이 EC2 Instance Profile의 access key로도 AWS Signiture Version 4가 될 거라 믿었지...
하지만 안. 된. 다.
InvalidAccessKeyId 라는 오류가 반환된다.


3. AWS defect인지, 아니면 S3 권한 문제인지.. 단순히 나의 실수인지 한참을 헤맸다.
그리고 EC2 Instance Profile의 access key는 temporary access key 라는 것을 알아냈다.
https://stackoverflow.com/questions/39920505/unable-to-access-s3-file-with-iam-role-from-ec2#comment67135836_39920505
결정적인 힌트

4. 위 댓글을 기반으로 AWS 문서를 확인해보니
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html#instance-metadata-security-credentials
To make a call outside of the instance using temporary security credentials (for example, to test IAM policies), you must provide the access key, secret key, and the session token.
라는 문구를 볼 수 있었다. (모르고 보면 잘 안보임)
session token이 필요하다는 것인데, 이 부분은 뒤에 이야기 하겠다.

5. 그리고 AWS Signature Version 4 의 다른 문서에도 다음과 같은 안내가 있다.
https://docs.aws.amazon.com/neptune/latest/userguide/iam-auth-connecting.html
If you are using temporary credentials, they expire after a specified interval, including the session token.
You must update your session token when you request new credentials. For more information, see Using Temporary Security Credentials to Request Access to AWS Resources.
바로 session token.
그리고 위 두 문서에서 공통적으로 안내하는 다음 문서를 보자

6. 여기서는 AssumeRole을 통해 SessionAWSCredentials이란걸 만드는데,
https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_use-resources.html
어쨋든 SDK를 활용해야 하는 나로써는 기존의 EC2 Instance Profile이 아닌 별도의 credentials - Session Credential이 필요하단걸 알게된다.


아하 AWS Signature Version 4에도 token을 넣을 수 있구나 -> 넣어야 하구나
https://docs.aws.amazon.com/AmazonS3/latest/API/RESTObjectPOST.html

자,
여기까지 정리하면,
EC2 Instance Profile로는 안된다. 왜냐하면 IAM User 와 달리 temporary 하기 때문이다.
(처음엔 EC2의 temporary말고 새로운 temporary가 필요한줄...)
Session token이 들어있는 Session Credentials가 필요하다.
그래야 token을 구하고, 파일을 업로드할 수 있다.
그러면 새로운 Session Credential은 어떻게 만드는가?
DefaultCredentialProviderChain을 이용해서 권한을 얻은다음 STS client로 만들면 된다.

https://stackoverflow.com/questions/18884683/browser-uploads-to-s3-with-instance-roles
좋은 힌트
STS는 필요없다?


EC2 Instance Profile로는 STS를 호출 할 수 없다. 오직 IAM User만 가능
https://stackoverflow.com/questions/35873012/error-when-ec2-running-as-a-role-tries-to-get-a-session-token-in-aws




결국 EC2 Instance Profile이 이미 훌륭한 temporary credential이므로 여기서 token을 구하자
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html#instance-metadata-security-credentials
Instance Metadata에 들어있다고 하네
이걸 어떻게 꺼내지?

찾아보니 EC2MetadataUtils 라는게 있네
거기서 .getIAMSecurityCredentials()를 하면 되는데, Map이라니.. key가 뭐지?
https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/util/EC2MetadataUtils.html
https://stackoverflow.com/questions/41287920/how-to-get-accesskey-secretkey-using-java-aws-sdk-running-on-ec2
돌려보면 key는 말그대로 role name이다. arn같은거 아닌 그냥 name이다




기타 참고
AWS의 Credential provider chain
https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/credentials.html
https://docs.aws.amazon.com/sdk-for-java/v2/developer-guide/credentials.html


Pre-signed Url을 얻는 방법(한글)
https://medium.com/@labcloud/s3-pre-signed-url-%EB%AF%B8%EB%A6%AC-%EC%84%9C%EB%AA%85%EB%90%9C-url-%EB%A7%8C%EB%93%A4%EA%B8%B0-596aff8bdc45
EC2 Instance Profile로 Pre-signed Url을 만드는데, 자세히 보면 Url의 Query param에 x-amz-security-token가 존재한다.

조금 상관없었지만, 어쨋든 실제 x-amz-security-token을 사용한다는 걸 알게된 글
https://github.com/minio/minio-go/issues/785

이걸 Java SDK로 구현하는 예제
https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/dev/AuthUsingTempSessionTokenJava.html
STS로 새로운 temporary access key를 만들면 되겠구나!

https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/dev/PresignedUrlUploadObjectJavaSDK.html
Java SDK로 pre-signed url을 만드는 방법, 이 역시 IAM User access key로 만들기 때문에 별로 도움은 안됨
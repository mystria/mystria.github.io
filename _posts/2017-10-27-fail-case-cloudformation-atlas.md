---
layout: post
title:  "금주의 AWS 실패사례 - CloudFormation으로 MongoDB Atlas와 연동 자동화"
date:   2017-10-27 16:00:00 +0900
categories: AWS CloudFormation MongoDB Atlas
comments: true
---
# 이번 주 AWS를 사용하면서 겪은 실패 사례

## 개요
  * 프로젝트에서 MongoDB Atlas를 사용 중. 간단하게 클릭 몇 번으로 클라우드 상에 DB Cluster를 구축할 수 있음.
    + 이 모든 기능이 RESTful API로 호출 가능
      - [MongoDB Atlas API Documents][mongodb-atlas-reference]
      - 단, 호출의 인증(Authentication)을 Digest&sup1;방법으로 하기 때문에 PostMan으로 시도해 볼 수 없었음&sup2;. 불길한 징조
    + 그래서 linux의 curl로 API 호출(Atlas Docs의 Example로도 제공)
      - 처음부터 curl쓸걸 괜히 PostMan으로 해보느라 시간 낭비함
      - curl(또는 cURL)은 다양한 protocol로 데이터를 전송하는 도구 : [curl][curl]
  * CloudFormation으로 생성가능한 영역은 AWS내부 resource뿐
    + CloudFormation으로 Atlas를 생성 후 AWS resource와 연동시키는 것은 불가능
    + 게다가 Atlas와 AWS의 VPC간 통신을 위해서는 VPC Peering이 필요
      - VPC Peering도 CloudFormation으로 설정 불가(정확히 말하면, CloudFormation으로 [request는 가능하지만][aws-cloudformation-vpc-peering] accept가 불가능)
      - VPC Peering이 없어도 통신할 수는 있지만 성능 저하 발생
  * 이런저런 복잡한 사정으로 CloudFormation단독으로는 어렵고 Python을 이용하기로 함
    + AWS의 API는 Python의 boto3 Library로 지원함
  * 삽질의 시작

## 구성방법 설계
  * 일단 MongoDB Atlas는 수작업으로 생성
    + Python으로 curl을 통해 API호출하여 클러스터를 생성해도 되지만,
      - 엄청난 traffic을 요구하거나 실제 운영을 위한 클러스터가 아니라면 클러스터 1개로 database이름만 바꿔서 여럿이서 사용하고자 함
      - 즉, 매번 생성할 필요가 별로 없음
      - [Python으로 curl호출 방법][python-call-curl]
  * CloudFormation으로 VPC및 Network 생성
    + VPC -> Route -> RouteTable -> Subnet 등
      - RouteTable에 Route를 만들어 넣고, RouteTable을 Subnet에 association하는 구조
      - 여기에 필요한 resource들도 추가로 생성
    + VPC ID, CIDR, RouteTable ID를 Output
  * Atlas API를 호출해 생성한 VPC와 Peering 요청
  * CloudFormation으로 Subnet에 association된 RouteTable에 VPC Peering용 Route 추가
  * AWS API를 호출해 Atlas의 VPC Peering 요청을 accept
  * 위 과정을 Python으로 구현

## MongoDB Atlas RESTful API
  * 공식 Docs에 충분한 예제들이 있음
  * 사용자 ID와 API Key는 우측 상단 Account정보(사용자 이름 클릭)에서 획득
  * MongoDB 사용자가 속해있는 Group의 ID 획득, Browser의 주소창에 URL로 표시됨  
    1. 현재 클러스터가 생성된 Container&sup3; 찾기. 필요한 값(사용자 ID, API Key, Group ID, *Region*) : [Docs][mongodb-atlas-get-container-id]
    2. VPC Peering 생성. 필요한 값(사용자 ID, API Key, Group ID, 현재 *Container ID*, AWS의 AccountID(숫자14자리), AWS의 *VPC ID*, VPC의 *CIDR*) : [Docs][mongodb-atlas-vpc-peering]
    3. 생성된 VPC Peering의 `Connection ID` 찾기. 필요한 값(2번에서 생성된 Peering ID) : [Docs][mongodb-atlas-get-connection]
      - 이때, Connection이 이뤄지는 중이라 Connection ID가 None으로 넘어 올 수 있음
      - 재시도 하면 됨
  * Response가 json이므로 Python의 json.loads()로 응답 처리
  * 다음과 같은 Error 발생
  ```
  curl: (35) gnutls_handshake() failed: Access was denied
  ```
    + Account정보에서 Public API Access의 API Whitelist에 개발환경의 CIDR을 추가해야 함

## AWS API
  * boto3 Library를 이용
  * [VPC Peering 요청을 accept][boto3-accept-peering]
    + 이때 위에서 얻은 Connection ID를 사용

## CloudFormation 설정
  * RouteTable 업데이트
    + 위에서 얻은 Connection ID를 target으로 하는 Route를 생성하고,
    + 필요한 Subnet의 RouteTable을 찾아 Route 추가
    + 이때, RouteTable ID를 Network 생성용 CloudFormation에서 Output으로 받아두는게 좋음

## 소감
뭔가 자동화 하려고 여러모로 발버둥 친 것 같은데, 자동화라기 보단 하드코딩이 된 것 같아서 불만.  
DB구성을 코드화 하는건 IaaC 개념을 위해 중요한 작업이지만, 여러 솔루션들을 이용하기 위해서는 이를 통합해서 제어할 수 있는 기술이 필요해 보인다.  
그리고 어디까지 자동화 해야 하는지 정책을 세워야 할 듯 하다.

---
&sup1; 간단히 말하면, 인증을 위해 사용자의 ID/PW를 REST의 Header에 담아서 보내면 노출 위험이 있므로, 서버로 부터 nonce값을 받아 hash하여 ID/PW를 인증받는 방식. nonce값은 첫번째 request의 response에 담겨서 옴.  
&sup2; 정확하게는 모르겠지만 본 작업 중에는 PostMan에서 Digest Auth를 지원하지 않았음. Digest용 form은 제공하는데 수작업으로 nonce값을 넣어야 헤서 인증이 성공하지 못함.  그런데 2017년 10월 초쯤 지원하게 된 듯([PostManLabs][postman-digest-authentication]).  
&sup3; MongoDB Atlas는 사용자용 VPC에 Container를 만들어 서비스, 이 Container는 Region별로 1개씩.  즉, Region별로 Container ID가 1개  

[postman-digest-authentication]: https://github.com/postmanlabs/postman-app-support/issues/3017
[curl]: https://curl.haxx.se/docs/manpage.html
[aws-cloudformation-vpc-peering]: http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-vpcpeeringconnection.html
[mongodb-atlas-reference]: https://docs.atlas.mongodb.com/configure-api-access/
[python-call-curl]: https://stackoverflow.com/questions/43975423/running-curl-command-in-python
[mongodb-atlas-vpc-peering]: https://docs.atlas.mongodb.com/reference/api/vpc-create-peering-connection/
[mongodb-atlas-get-container-id]: https://docs.atlas.mongodb.com/reference/api/vpc-get-containers-list/
[mongodb-atlas-get-connection]: https://docs.atlas.mongodb.com/reference/api/vpc-get-connection/
[boto3-accept-peering]: http://boto3.readthedocs.io/en/latest/reference/services/ec2.html#vpcpeeringconnection

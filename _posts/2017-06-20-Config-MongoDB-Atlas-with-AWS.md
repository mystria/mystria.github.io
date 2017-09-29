---
layout: post
title:  "MongoDB Atlas와 AWS를 연계하여 구축하기"
date:   2017-06-20 12:00:00 +0900
categories: AWS MongoDB Atlas VPC Migration Robomongo
comments: true
---
# MongoDB Atlas를 설치(?)하며 진행한 절차 정리
  
## MongoDB Atlas란?
 * 공식홈페이지 참조 [MongoDB Atlas][official-mongodb-site]
 * 개인적인 정리로는 관리형 MongoDB 서비스라고 보면 될 듯
 * 원래라면 MongoDB 설치를 위해 서버를 구축하고 traffic, scale in-out, monitoring 등 관리할게 많으나 이를 Atlas가 대신 해줌. 서버까지 임대해주는 개념
 * AWS외에 MS Azure, Google GCP도 지원하지만, 본 글은 AWS 중심으로 설명
  
## 설치 순서(AWS 구축과 함께)
 1. MongoDB 회원가입
 2. MongoDB Atlas Group 및 Cluster생성 (여기서 Cluster가 DB서버들이라고 보면 됨)
    * 무료는 크기가 적고, 안되는게 많지만 실습용으로 적합
    * username과 password 확인
 3. AWS에서 VPC생성
    * Subnet등 구성요소도 같이 생성
    * 주의사항: 향후 VPC Peering을 위해 CIDR중복 방지
 4. MongoDB Atlas에서 VPC Peering 신청
 5. VPC에서 Peering동의하고 route table설정 (VPC -> Atlas)
 6. VPC용 Security Group 생성
 7. Atlas에서 Peering완료 확인 후 Security Group으로 Whitelist처리
 이를 통해 VPC와 Atlas간 연결이 확보되었으므로, VPC내 EC2 instance들이 접근해서 사용하면 끝.
  
## MongoDB 데이터 Export / Import 하기
 * Collection단위로 파일로 export/import가 됨(좀 더 확인이 필요)
  
### Export
 1. 기존 MongoDB가 설치된 EC2 instance접속
    1. MongoDB는 보통 Private Subnet에 있고 SSH로 접속해야 할 수 있음
    2. SSH를 통해 NAT instance에 접속
        * 이때 ssh-add -K key.pem을 하여 key forwarding
    3. NAT instance에서 MongoDB instance로 SSH로 접속
    * 사실 SSH tunneling을 쓰면 되는거 같은데, 복잡하니까 이렇게..
 2. mongoexport 사용
    * 다음 매뉴얼 참고 [Export Manual][official-mongodb-doc-export]

    ```bash
    $ mongoexport --db *database* --collection *collection_name* --out *collection_name.json*
    ```

 3. json형식 등으로 export 후 S3로 업로드(복사)

    ```bash
    $ aws s3 cp ~/export s3://bucket --recursive
    ```

 4. export된 파일 다운로드
  
### Import
 * MongoDB instance가 private subnet이므로 바로 import시킬 수 없음
 * Local 컴퓨터에 MongoDB를 설치하기를 추천
 * mongoimport 사용
    * 다음 매뉴얼 참고 [Import Manual][official-mongodb-doc-import]

    ```bash
    $ mongoimport --host *host_address:27017* --db *database* --type json --file *~/Downloads/collection_name.json* --authenticationDatabase admin --ssl --username *username* --password *password*
    ```
    
  
## Robomongo로 확인하기
 * Robomongo란? [Robomongo][robomongo]
 * MongoDB Atlas는 TLS/SSL을 요구하며 MongoDB 3.4를 지원함
 * Robomongo도 업데이트를 통해 TLS/SSL를 지원 [rc10][robomongo-rc10]
 * MongoDB 3.4도 지원 [3T][robomongo-3t]
 * 또한 최신 업데이트로 Replica Set조회도 가능(Atlas 지원을 위함인듯) [rc1][robomongo-1-rc1]
  
 1. Robomongo 3T를 설치하고, Connection 생성 시 type을 Replica Set으로 설정
 2. Authentication에 database는 admin(무조건)으로 하고, username과 password(SCRAM-SHA-1)입력
 3. SSL을 사용하게 바꾸고 인증방식을 Self-signed Certificate로 선택
  
[official-mongodb-site]: https://www.mongodb.com/cloud/atlas
[official-mongodb-doc-export]: https://docs.mongodb.com/manual/reference/program/mongoexport/
[official-mongodb-doc-import]: https://docs.mongodb.com/manual/reference/program/mongoimport/
[robomongo]: https://robomongo.org/
[robomongo-3t]: http://blog.robomongo.org/robomongo-is-robo-3t/
[robomongo-rc10]: http://blog.robomongo.org/robomongo-rc10/
[robomongo-1-rc1]: http://blog.robomongo.org/robomongo-1-rc1/


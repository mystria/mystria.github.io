---
layout: post
title:  "금주의 실패사례 - 망가진 MongoDB ReplicaSet 살리기"
date:   2018-02-04 05:00:00 +0900
categories: MongoDB AMI
comments: true
---
# MongoDB의 ReplicaSet이 망가졌다
개인적으로 SaaS를 통해 DB든 모니터링이든 편하게 이용하고 싶은데, 비용적인 문제, 보안 문제 등으로 인해 직접 설치해서 사용해야하는 경우가 있다.  
MongoDB 같은 경우 ReplicaSet을 구성하여 이용하는 경우가 많은데, 어느날 이 중 하나가 망가져 있었다.  
ReplicaSet 중 하나가 망가지니 나머지에 load가 몰려 전체적인 throughput이 떨어지는 문제가 발생 했다.  
이 글은 AWS EC2에서 돌아가던 MongoDB ReplicaSet을 살리려 노력했던 날의 기록이다.  

## MongoDB 상태 이상 확인
* CloudWatch의 Dashboard에 이상이 감지됨
  + 3대의 ReplicaSet중 한 대는 CPU가 100%인데, 한 대는 0%인 것
* 문제 원인 찾기
  + VPC, Subnet, SecurityGroup 각 설정을 확인해봤지만 이상 없음
    - 애초에 잘 되던게 갑자기 안되는거니, 누군가 설정에 손댄게 아니라면 잘 되겠지...
  + EC2의 모니터링 상태는 정상
    - Instance의 문제가 아니라 MongoDB 자체의 문제?
* Primary에 접근하여 ReplicaSet 확인
  + 각 ReplicaSet으로 접속 시도 : 정상 member
  ``` sh
  $ mongo --host 주소 --port 27017
  MongoDB shell version: 3.4.10
  connecting to: 주소:27017
  >
  ```
  + 각 ReplicaSet으로 접속 시도 : 이상있는 member
  ``` sh
  $ mongo --host 주소 --port 27017
  warning: Failed to connect to 주소:27017, reason: errno:111 Connection refused
  blahblah..
  ```
  + Primary의 mongo console에 접속
  + rs.status() 실행
    - 문제가 있던 ReplicaSet member의 상태가 "unavailable"로 표시됨
    - 사실 정확히 어떤 메시지 였는지 기억이 안남
    - "unreached", "disconnected" 뭐 이런 종류였음  
  + 이상있는 member에 왜 접근이 안될까?
    - 네트워크 문제?
    - mongo 서비스 문제?
* Member에 접근하여 확인
  + mongo console 접속
    - 안됨
  + mongo 서비스 확인 : [참고](https://stackoverflow.com/questions/5091624/is-mongodb-running)
  ``` sh
  $ ps -edaf | grep mongo
  ```
    - 없음
  + MongoDB 서비스가 죽은것이 원인!

## MongoDB 살리기
* 서비스 다시 실행 시키기
  + 참고 [[MongoDB process 시작/종료](https://docs.mongodb.com/manual/tutorial/manage-mongodb-processes/)]
  + mongod.config 확인
  + systemctl 확인 : mongodb daemon을 service로 등록 [[참고](https://askubuntu.com/questions/748789/run-mongodb-service-as-daemon-of-systemd-on-ubuntu-15-10)]
  ``` sh
  $ systemctl status mongod
  ```
  + 서비스 시작
  ``` sh
  $ sudo service mongod start
  ```
  + Exit code 14를 반환하면서 실패
    - Code 14란? Unrecoverable error [[참고](https://docs.mongodb.com/manual/reference/exit-codes/)]
* MongoDB 복구하기
  + Unrecoverable은 뭘까?
    - 직감적으로 MongoDB의 dbpath에서 mongod.lock 확인 : 이전 PID가 기록되어 있음
  + Repair
    - 한글 참고 : [mongod.lock 관련 에러 해결하기](http://sungpil.com/2015/03/19/euaeu/)
    - StackOverflow들 : [mongodb service is not starting up
](https://stackoverflow.com/questions/9884233/mongodb-service-is-not-starting-up), [MongoDB won't start after server crash
](https://stackoverflow.com/questions/13700261/mongodb-wont-start-after-server-crash)
  ``` sh
  $ sudo mongod --dbpath /var/lib/mongodb --repair
  ```
  + 시간은 2시간씩 걸리는데 제대로 복구가 안됨
    - 서비스를 시작시 Exit code 100 (Unknown Exception) 발생
* 시간은 시간대로 낭비되는데 정확한 원인도 찾을 수 없어 포기

## ReplicaSet 복제 추가
* 정상 member를 복제한 후 ReplicaSet에 추가
  + 바보같은 실패 케이스
    1. 정상 instance에서 MongoDB의 dbpath로 설정된 EBS를 snapshot
    2. Snapshot을 EBS로 생성
    3. 정상 instance를 선택 후, Actions > Launch More Like This
    4. 새로 생성된 Instance에 생성된 EBS attach
    5. Instance에 SSH로 접속 후 EBS mount : [참고](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/ebs-using-volumes.html)
    6. 하다보니 뭔가 이상...
    7. "Launch More Like This" 는 해당 instance와 동일한 AMI를 동일한 설정값으로 생성하는 것
      - MongoDB가 설치되어 있지 않고, 각종 설정파일들이 세팅되어 있지 않음
      - 이것은 Chef로 설정되어 있던 것!
      - 사실 이 MongoDB는 다른 조직이 설치 후 인수인계 받은 것이며, Chef는 사라졌음(인수인계 못받음ㅠㅜ)
    8. 빠른 포기 후 instance와 EBS, snapshot 삭제
  + 성공 케이스
    1. 정상 instance를 선택 후, Actions > Image > Create Image
    2. 정상 instance를 선택 후, Actions > Launch More Like This
    3. AMI는 방금 생성한 image로 변경
    4. 이때, SecurityGroup설정이 초기화 되므로, 기존 SG선택
    5. 생성 완료 후, Primary에 접속 mongo console 실행
    6. rs.add(호스트주소:27017) 실행 : [참고](https://docs.mongodb.com/manual/tutorial/expand-replica-set/)
    7. rs.remove(기존member의호스트주소:27017) 실행 : [참고](https://docs.mongodb.com/manual/tutorial/remove-replica-set-member/)
    8. MongoDB설정이 들어가면서 DB Read/Write가 잠깐 멈추는 듯
    9. rs.status()로 확인
* Application에 ReplicaSet 주소를 변경
  + 원래 App의 (Java일 경우) JVM설정 등을 변경하여 신규 ReplicaSet을 연결해야 함
  + 이번 케이스의 경우 해당 ReplicaSet이 Route53 Record로 등록되어 있었음
    - Route53의 Record를 신규 ReplicaSet member로 변경
* CloudWatch로 확인
  + 정상동작 확인!!

## 평가
* 불변 인프라
  + 서버가 24/7 잘 동작하는게 최고겠지만, 어찌됐건 서버가 죽을 수도 있는 일이다.  
  + 망가진 서버를 고쳐서 복구시키는 것도 방법이겠지만, 복구를 위해 소요되는 비용, 복구과정에서 알게 모르게 변경되는 여러 설정들을 생각하자면,
  + 깔끔하게, 그리고 동일하게 새로 만드는 것이 여러모로 낫다.  
  + 그러기 위해선 코드로 정의된 인프라(Infrastructure as Code)와 데이터 일관성(Consistency), 끊김없는 전환(Seamless)이 필요한 것이다.  
  + 업데이트가 아닌 복구(재해 복구) 측면에서도 불변 인프라(Immutable Infrastructure)는 유효하다고 생각된다.
* 그런데 MongoDB는 왜 깨졌을까?
  + 영원히 알 수 없게 되버림...

## 참고
[MongoDB Replica Set 구성하기](http://minsql.com/blog/mongodb-replica-set-%EA%B5%AC%EC%84%B1%ED%95%98%EA%B8%B0/)

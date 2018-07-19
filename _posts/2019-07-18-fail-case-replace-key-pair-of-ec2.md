---
layout: post
title:  "금주의 AWS 실패사례 - "
date:   2018-07-18 20:00:00 +0900
categories: AWS EC2
comments: true
---
# EC2의 key pair를 바꿔보자
Amazon EC2에 SSH로 접속하기 위해서는 key pair가 필요하다. EC2 instance를 생성할 때 이 key pair를 생성 또는 지정이 가능하다.  
사용자는 private key를 다운로드 받을 수 있는데, 이 key파일(.pem)을 분실 할 경우, 해당 key의 EC2 instance는 더 이상 접근이 불가능하다.  
이 key를 분실하였을때 대처 방법이 AWS 공식으로 제공되니 확인해보자.

## 공식 해결책
  * 프라이빗 키를 분실했을 때 Linux 인스턴스에 연결하는 방법
    + https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/ec2-key-pairs.html#replacing-lost-key-pair
  * 요약
    + 분실한 instance의 EBS를 분리
    + 다른 임시 instance에 해당 EBS를 부착
    + .ssh/authorized_keys 파일을 덮어 씀
    + 다시 해당 EBS를 원래 instance에 부착

## 실제로 했을 때 trouble shooting
  * EBS를 임시 instance에 붙인 후 mount가 되지 않음
    + Error 1: 잘못된 파티션 선택
    ~~~
      mount: /dev/xvdf1 is write-protected, mounting read-only
      mount: you must specify the filesystem type
    ~~~
      - 경우에 따라 부착한 EBS가 DISK일 수도 있고 PART일 수도 있음
      - lsdsk로 확인하면 TYPE이 표시됨
      - 이 때 파티셔닝된 EBS일 경우 파티션(뒤에 숫자가 붙어있음)을 제대로 mount 해야함
      - 참고: [(Stack Overflow)Attaching and mounting existing EBS volume to EC2 instance filesystem issue](https://stackoverflow.com/questions/28792272/attaching-and-mounting-existing-ebs-volume-to-ec2-instance-filesystem-issue)
    + Error 2-1: mount 시 뭔가 에러
    ~~~
      mount: wrong fs type, bad option, bad superblock on /dev/xvdf2,
             missing codepage or helper program, or other error

             In some cases useful info is found in syslog - try
             dmesg | tail or so.
    ~~~
      - 제대로 된 파티션을 지정했지만, 뭔가 잘못되었다고 함
      - $ dmesg | tail 을 실행해보면 로그가 표시됨
      - 참고: [(ServerFault)Cannot mount an existing EBS on AWS](https://serverfault.com/questions/632905/cannot-mount-an-existing-ebs-on-aws)
      - Error 1과 Error 2는 사실상 비슷한 이유
    + Error 2-2: duplicate UUID
    ~~~
      Filesystem has duplicate UUID 6785eb86-c596-4229-85fb-4d30c848c6e8 - can't mount
    ~~~
      - 로그를 살펴보면 중복된 UUID라고 함 (같은 AMI 기반으로 생성해서 그런가?)
      - UUID를 무시하고 mount하면 됨
      - 참고: [XFS Filesystem has duplicate UUID problem](https://linux-tips.com/t/xfs-filesystem-has-duplicate-uuid-problem/181)

## 변경에 성공
  * 잘 안될 것 같지만 생각보다 잘 됨
  * 단, 안전을 위해 snapshot을 미리 만들어 둘 것
  * key가 변경되더라도 EC2 instance의 description에서는 Key pair name이 그대로 유지됨
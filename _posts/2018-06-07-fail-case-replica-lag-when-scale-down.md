---
layout: post
title:  "Replica Lag 줄이기 - Instance의 Scale-down"
date:   2018-06-12 22:00:00 +0900
categories: AWS RDS Database
comments: true
---
# 잘 안쓰이는 RDS의 크기를 줄였더니 Replica Lag 발생
Amazon RDS의 MySQL 서버 중에 데이터 분석을 위한 Read Replica가 있었다. 주기적으로 데이터 분석을 위해 30분에 한 번 정도 조회하는 용도로만 사용될 뿐. 이 DB 서버의 크기가 불필요하게 큰 것 같아서 절반 정도로 줄였더니 Replica Lag이 600000ms(10분)를 넘어서는 문제가 발생했다.

## 원인
  * Replica Lag(복제 지연)이란?
    + Oracle의 설명
      > Replication lag occurs when the slaves (or secondaries) cannot keep up with the updates occuring on the master (or primary). Unapplied changes accumulate in the slaves' relay logs and the version of the database on the slaves becomes increasingly different from that of the master.

    + 복제 지연은 슬레이브(또는 보조)가 마스터(또는 기본)에서 발생하는 업데이트를 따라잡을 수 없을 때 발생합니다. 적용되지 않은 변경 사항은 슬레이브의 릴레이 로그에 누적되며, 슬레이브에 대한 데이터베이스 버전은 마스터와의 차이는 점점 커집니다.
      - 즉, 슬레이브의 마스터 복제가 지연되는 것
    + [What Causes Replication Lag?](https://blogs.oracle.com/jsmyth/what-causes-replication-lag)
  * Instance Type 변경
    + 기존 instance 크기에서 데이터 조회 할 때 마다 CPU 점유율이 30% 정도
    + Instance 크기를 절반으로 줄였더니 CPU 점유율이 80~100% 으로 증가
    + CPU가 점유되는 동안(데이터 조회 중) replication이 되지 않아 replica lag 발생
    + Replica lag 발생시 마다 CloudWatch 알람 발생
    + 조회가 끝나고 시간이 지나면 replica lag은 0으로 정상화

## 해결책
  * innodb_flush_log_at_trx_commit를 1(기본)에서 2로 변경
    + 즉, 매 트랜젝션 당 flush하는 것을 약 1초에 한 번 하는 것으로 변경
    + 출처: [MySQL InnoDB Replication Lag 줄이기](http://blog.recopick.com/46)
  * tx_isolation을 repeatable read(기본)에서 read committed로 변경
    + 즉, Isolation Level을 변경
    + 이런 저런 이유로 transaction이 걸렸을 때도 읽기는 가능하게 변경
    + 출처: [Fixing replica lag on MySQL slaves](https://discourse.looker.com/t/fixing-replica-lag-on-mysql-slaves/787)
    + 출처: [MySQL 트랜잭션 Isolation Level로 인한 장애 사전 예방 법](http://gywn.net/2012/05/mysql-transaction-isolation-level/)

## Amazon RDS에서는?
  * tx_isolation을 변경하기 위해서는 DB의 설정을 바꿔야 함
  * RDS에는 Parameter Group이란 것이 존재
    + 선택한 RDS instance의 description에 보면 Parameter Group이 있음
    + 여기 있는 값들을 수정하면 DBMS의 속성을 제어 가능함(때론 서버 재시작 필요)
    + In Sync일 경우 설정이 서버에 반영된 상태, Applying은 적용 중
    + 참고: [DB 파라미터 그룹 작업](https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/UserGuide/USER_WorkingWithParamGroups.html)

## 결론
  * Parameter Group에서 tx_isolation을 read committed로 변경하여 해결
  * 찾아보면 다양한 해결책이 있을 듯

---
layout: post
title:  "MongoDB Atlas Manual Migration 방법"
date:   2018-01-29 13:00:00 +0900
categories: MongoDB Migration Import
comments: true
---
# MongoDB Atlas로의 수동 Migration(Import) 방법

## Legacy(?) MongoDB에서 MongoDB Atlas로 이전
- 서버에서 동작 중인 MongoDB(개발자가 직접 설치)의 데이터를 MongoDB Atlas로 옮겨야 하는 일이 생김
- MongoDB Atlas에서 지원하는 Live Migration을 시도하고자 하였으나 database이름이 중복되는 문제 발생
- 확인을 해보진 못했으나, 상식적으로 좋지않은 일이 생길 것이라고 직감
- 기존 MongoDB의 database이름을 바꾸고 Migration 해보자!

## 준비사항
  - 기존 서버와 MongoDB Atlas간 방화벽 체크
  - AWS일 경우
    - VPC Peering
    - Route Table
    - IP Whitelist
  - MongoDB Atlas에 접속할 ID/Password 확인

## 진행순서
1. Connect an origin MongoDB shell
1. Access the mongodb console  
```$ mongo```
1. Rename the database if you want  
```> db.copyDatabase('inventory','new_inventory');
{ "ok" : 1 }```
1. Stop database writes to your existing database  
```> db.fsyncLock();{
        "info" : "now locked against writes, use db.fsyncUnlock() to unlock",
        "seeAlso" : "http://dochub.mongodb.org/core/fsynccommand",
        "ok" : 1
}```
1. This works can be applied on shell like this  
```$ mongo --eval "db.copyDatabase('inventory','new_inventory')"
$ mongo --eval "db.fsyncLock()"```
1. Exit the mongodb console
1. Dump original data  
```$ mongodump --db=new_inventory```
1. Check to connect the target mongodb server  
```$ ping cluster-dev-shard-00-00-z35un.mongodb.net```
1. Restore dumpped data to the target MongoDB  
```$ mongorestore --host Cluster-shard-0/cluster-shard-00-00-abcde.mongodb.net:27017,cluster-shard-00-01-abcde.mongodb.net:27017,cluster-shard-00-02-abcde.mongodb.net:27017 --ssl --username <username> --password <password> --authenticationDatabase admin ~/dump/```
1. Unlock database writes
```$ mongo --eval "db.fsyncUnlock()"
{ "info" : "unlock completed", "ok" : 1 }```

## Reference
* [MongoDB Atlas Migration]( https://www.mongodb.com/blog/post/migrating-data-to-mongodb-atlas)  
* [MongoDB 백업하고 복구하기](https://blog.outsider.ne.kr/790)  
* [How do you rename a MongoDB database?](https://stackoverflow.com/questions/9201832/how-do-you-rename-a-mongodb-database)  
* [Document of mongorestore](https://docs.mongodb.com/manual/reference/program/mongorestore/)

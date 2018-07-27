---
layout: post
title:  "MongoDB Atlas Manual Migration 방법 - 2편"
date:   2018-07-27 17:00:00 +0900
categories: MongoDB Migration
comments: true
---
# MongoDB Atlas로의 수동 Migration(mongodump - mongorestore) 방법

## Legacy(?) MongoDB에서 MongoDB Atlas로 이전
- 전에 동일한 이슈로 글을 작성: [MongoDB Atlas Manual Migration 방법]({% post_url 2018-01-29-mongodb-manual-migration %})
- EC2 instance에서 동작 중인 MongoDB(개발자가 직접 설치)의 데이터를 MongoDB Atlas로 옮겨야 하는 일이 생김
- MongoDB Atlas에서 지원하는 Live Migration을 시도하고자 하였으나 database이름이 중복되는 문제 발생
- 확인을 해보진 못했으나, 데이터가 한 database에 몰릴 것으로 추정
- 기존 MongoDB의 database를 dump 후 새 database 이름으로 restore 해보자!
- 데이터 크기가 크지 않을 때 적절함

## 준비사항
  - 기존 서버(EC2 instance)와 MongoDB Atlas간 방화벽 체크
    - VPC Peering
    - Route Table
    - IP Whitelist
  - MongoDB Atlas에 접속할 ID/Password 확인

## 진행순서
1. Connect the origin server(EC2 instance's ssh)
1. Create a dump files using "mongodump" operation
``` ssh
$ mongodump -d old_db_name -o mongodump/
```
1. MongoDB Atlas requires to be connected via SSL, use "--ssl" option
1. Authenticate using "-u --username, -p --password, --authenticationDatabase" options
1. Rename the database using "-d" option if you want
1. The last parameter is dumped directory that contains "Collections" json files  
1. Run restoration to the target MongoDB Atlas cluster
``` ssh
$ mongorestore --ssl --host cluster-primary-node.mongodb.net --port 27017 -u <username> -p <password> -d new_db_name --authenticationDatabase admin mongodump/old_db_name
```
1. Done. It's simpler.

## Reference
* [Migrating Data to MongoDB Atlas](https://www.mongodb.com/blog/post/migrating-data-to-mongodb-atlas)  
* [MongoDB Documents - mongorestore](https://docs.mongodb.com/manual/reference/program/mongorestore/)  

---
layout: post
title:  "금주의 AWS 실패사례 - Python3과 Python2의 S3 GET_OBJECT 차이"
date:   2018-02-26 17:00:00 +0900
categories: AWS Python S3
comments: true
---
# Python을 이용해 S3에서 파일 읽기
S3에 위치하고 있는 파일을 읽어야 하는 Python script를 개발해야한다.  
Python2에서는 잘 되던게 Python3에서는 에러를 낸다면?  

## S3 Get Object
- S3는 Object storage로 file이라는 개념과는 조금 다름
- 어쨋든 S3에 위치한 파일은 Bucket과 Object라는 식별자로 읽어와야 함
- Python으로 S3를 접근하기 위해서는 Boto3 library를 이용해야 함

## Boto3 library
- Python의 Boto3을 이용해 S3에 위치한 json 파일을 읽어오기
- Sample
''' Python
import boto3
import json
...
s3 = boto3.client("s3")
bucket = "test_bucket"
key = "test_object_key"
obj = s3.get_object(Bucket=bucket, Key=key)
json_value = json.dumps(json.load(obj['Body']))
'''

## Python2와 Python3
- 위 sample처럼 하면 Python2에서는 정상 동작함
- S3에서 가져온 Object는 이미 json형식인데, 그 중 'Body'항목을 가져와야 함
- Body항목은 다음과 같은 형식: <botocore.response.StreamingBody object at 0x5f173a57dfd0>
- Python3에서는 다음과 같은 에러 발생
'''
TypeError: the JSON object must be str, not 'bytes'
'''
- 같은 library인데, 왜 str과 bytes로 달라지는 지 이유는 정확히 알 수 없지만, 해결책은 찾음. (제보 요망)
- Python3에서 실행 시 다음과 같이 해결
''' Python
json_value = json.dumps(json.loads(obj['Body'].read().decode('utf-8')))
'''
- Object의 'Body'가 bytes로 인식되므로 이를 read() 하고 utf-8로 decode하여 String으로 만듦
- String을 json.load하기 위해선 loads() 함수 이용
- Python2에서도 이용 가능

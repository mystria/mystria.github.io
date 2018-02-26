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
key = "test_object_key.json"
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
  + Python2의 경우 Object가 unicode임
  '''
  {u'Body': <botocore.response.StreamingBody object at 0x...>, u'AcceptRanges': 'bytes', u'ContentType': 'binary/octet-stream', 'ResponseMetadata': {'HTTPStatusCode': 200, 'RetryAttempts': 0, 'HostId': '...', 'RequestId': '...', 'HTTPHeaders': {'content-length': '13279', 'content-encoding': 'UTF-8', 'x-amz-id-2': '...', 'accept-ranges': 'bytes', 'server': 'AmazonS3', 'last-modified': 'Tue, 20 Feb 2018 10:54:57 GMT', 'x-amz-request-id': '...', 'etag': '"..."', 'date': 'Fri, 23 Feb 2018 07:19:31 GMT', 'x-amz-version-id': '...', 'content-type': 'binary/octet-stream'}}, u'LastModified': datetime.datetime(2018, 2, 20, 10, 54, 57, tzinfo=tzutc()), u'ContentLength': 13279, u'ContentEncoding': 'UTF-8', u'VersionId': '...', u'ETag': '"..."', u'Metadata': {}}
  '''
  + Python3의 경우 Object가 UTF-8임
  '''
  {'VersionId': '...', 'ContentType': 'binary/octet-stream', 'ContentEncoding': 'UTF-8', 'Body': <botocore.response.StreamingBody object at 0x...>, 'Metadata': {}, 'ResponseMetadata': {'HTTPHeaders': {'last-modified': 'Tue, 20 Feb 2018 10:54:57 GMT', 'server': 'AmazonS3', 'x-amz-request-id': '...', 'etag': '"..."', 'date': 'Fri, 23 Feb 2018 07:19:50 GMT', 'content-type': 'binary/octet-stream', 'accept-ranges': 'bytes', 'content-length': '13279', 'x-amz-id-2': '...', 'x-amz-version-id': '...', 'content-encoding': 'UTF-8'}, 'RetryAttempts': 0, 'HostId': '...', 'RequestId': '...', 'HTTPStatusCode': 200}, 'AcceptRanges': 'bytes', 'ETag': '"..."', 'ContentLength': 13279, 'LastModified': datetime.datetime(2018, 2, 20, 10, 54, 57, tzinfo=tzutc())}

  '''
- Python3에서 실행 시 다음과 같이 해결
''' Python
json_value = json.dumps(json.loads(obj['Body'].read().decode('utf-8')))
'''
- Object의 'Body'가 bytes로 인식되므로 이를 read() 하고 UTF-8로 decode하여 String으로 만듦
- String을 json.load하기 위해선 loads() 함수 이용
- Python2에서도 이용 가능

## 참조
- 해결책: https://stackoverflow.com/questions/31976273/open-s3-object-as-a-string-with-boto3/35376156
- json: https://www.slideshare.net/dahlmoon/json-20160301
- 기타: https://stackoverflow.com/questions/956867/how-to-get-string-objects-instead-of-unicode-from-json

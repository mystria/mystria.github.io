---
layout: post
title:  "Mac에서 AWS CLI 버전 업그레이드 하기"
date:   2020-02-08 01:00:00 +0900
categories: AWS Mac
comments: true
---
# MacOS의 AWS CLI 업그레이드
설치 방법은 [AWS 가이드](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/install-macos.html)를 따라하면 되지만, 업그레이드는?  
간단할 것 같지만 이상하게도 업그레이드가 안된다.  

## aws --version
* 버전 확인
  + 낮은 버전(Cli도 1.9.7인데, Python도 2.7)이 나옴
  ~~~ sh
    $ aws --version
    aws-cli/1.9.7 Python/2.7.16 Darwin/19.2.0 botocore/1.5.83
  ~~~
  + 업그레이드를 시도해도 up-to-date 하거나 satisfied 하다며 자꾸 skip됨
  ~~~ sh
    $ pip3 install awscli --upgrade --user
    Requirement already up-to-date: awscli in ./Library/Python/3.6/lib/python/site-packages (1.17.12)
    Requirement already satisfied, skipping upgrade: s3transfer<0.4.0,>=0.3.0 in ./Library/Python/3.6/lib/python/site-packages (from awscli) (0.3.3)
    ... 이하 생략
  ~~~
  + Python3 버전과 pip버전이 낮아서 그렇구나!
    1. Python3 업그레이드 
      - brew upgrade python3
    2. pip3 업그레이드
      - pip3 upgrade pip 
    3. awscli 업그레이드
      - sudo pip3 install awscli --upgrade --user
  + 다운로드 & 설치되는 화면이 막 나옴, 다 됐을까?
  ~~~ sh
    $ aws --version
    aws-cli/1.9.7 Python/2.7.16 Darwin/19.2.0 botocore/1.5.83
  ~~~
    - 그대로... 뭐가 문제인가?
  + 더불어, pip3 가 업그레이드 되면서 기존 버전 pip 실행이 안됨
    - /usr/local/bin/pip: bad interpreter: /usr/local/opt/python/bin/python3.6: no such file or directory
    - 이건 나중에 고치자...

## 기존 버전을 지우고 pip3로 새로 받으라
* 검색 결과 기존 awscli를 지우는게 좋다고 함
  + https://stackoverflow.com/questions/48572523/how-to-uninstall-aws-cli
  + https://stackoverflow.com/questions/47146493/aws-cli-is-using-python-2-instead-of-python-3 (같은 문제지만 해답 없음)
* 기존의 awscli를 지워보자
  + $ which aws 를 통해 옛 버전 aws의 절대 경로를 찾음
  + /usr/local/bin/aws
  + 앗! 이것은 (오래되어 까먹은) 옛날에 python2.7 기반으로 번들로 설치한 흔적
  + [공식 가이드](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/install-bundle.html#install-bundle-uninstall) 대로 작업
  ~~~ sh
    $ sudo rm -rf /usr/local/aws
    $ sudo rm /usr/local/bin/aws
  ~~~
* 삭제 후 기존 aws는 동작 안함
* 근데, pip3 로 설치한 aws도 동작 안함
  + Python3 에 설치된 aws경로를 export를 이용해 PATH에 등록해야 한다고 함
    - .bashrc 나 .bash_profile, .profile 등을 수정해야 할지도...
  + 귀찮은데...

## 새 번들을 받아서 재설치
* 그냥 번들 설치 관리자 방식으로 설치
  + https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/install-bundle.html
* Python2는 더 이상 지원되지 않으므로 Python3 기반으로 설치해야 함
  + which python3 로 python3 절대 경로 찾기
  + 절대 경로를 이용해 번들 설치 
  ~~~ sh
    $ which python3
    /usr/bin/python3
    $ sudo /usr/bin/python3 awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
    Running cmd: /Library/Developer/CommandLineTools/usr/bin/python3 -m venv /usr/local/aws
    Running cmd: /usr/local/aws/bin/pip install --no-binary :all: --no-cache-dir --no-index --find-links file://. setuptools_scm-3.3.3.tar.gz
    Running cmd: /usr/local/aws/bin/pip install --no-binary :all: --no-cache-dir --no-index --find-links file://. wheel-0.33.6.tar.gz
    Running cmd: /usr/local/aws/bin/pip install --no-binary :all: --no-build-isolation --no-cache-dir --no-index  --find-links file:///Users/UserName/awscli-bundle/packages awscli-1.17.12.tar.gz
    Symlink already exists: /usr/local/bin/aws (기존에 설치된게 남아있으면 나타남)
    Removing symlink.
    You can now run: /usr/local/bin/aws --version
  ~~~
  + /usr/local/bin 은 보통 PATH에 포함되어 있음
* 업그레이드 완료
  + 확인
  ~~~ sh
    $ aws --version
    aws-cli/1.17.12 Python/3.7.3 Darwin/19.2.0 botocore/1.14.12
  ~~~
  + aws-cli 가 업그레이드 되고, 연계된 Python 버전도 변경됨

## 결론
번들 설치 관리자로 설치했으면, 번들 설치 관리자로 업그레이드 해야함

---
layout: post
title:  "keystore로 받은 인증서에서 pem파일을 뽑아보자"
date:   2018-02-02 17:00:00 +0900
categories: Certificate
comments: true
---
# keystore로 받은 인증서를 사용해보자
SSL인증서가 필요하여 인증기관으로부터 인증서를 구매했다.  
난 pem파일로 된 인증서가 필요한데, 인증기관에서 keystore파일을 보내줬다면?!  
다시 인증기관에게 연락해서 pem파일로 다시 달라고 하면 되겠지만,  
아마추어처럼 보이니 직접 키를 뽑아내보자.  

## keystore 파일이란?
- 확실하진 않지만, wildcard인증서를 Tomcat기반으로 수령하게 되면 keystore형식으로 주는 듯 함
- Java SDK와 함께 배포되는 keytool로 관리되는 일종의 key database [[참고](http://btsweet.blogspot.sg/2014/06/tls-ssl.html)]
- 다음과 같이 JRE를 설치해도 사용가능
``` sh
$ sudo apt-get install openjdk-8-jre-headless
```

## keystore 파일을 p12파일로..
- 한글 정보 : [\[SSL\] Keystore에서 PEM 추출하기](https://medium.com/@jinro4/ssl-keystore%EC%97%90%EC%84%9C-pem-%EC%B6%94%EC%B6%9C%ED%95%98%EA%B8%B0-20cf49102283)
- 아래와 같이 keytool을 이용하여 p12파일 추출(추출 암호와 생성 암호를 동일하게 지정할 것)
``` sh
$ keytool -importkeystore -srckeystore [KEYSTORE] -destkeystore keystore.p12 -deststoretype PKCS12
Importing [KEYSTORE] keystore to keystore.p12...
Enter destination keystore password:  
Re-enter new password:
Enter source keystore password:  
Entry for alias ca successfully imported.
Entry for alias chain successfully imported.
Entry for alias alias successfully imported.
Import command completed:  3 entries successfully imported, 0 entries failed or cancelled
```
- 이렇게 p12파일이 생성되면, GUI단의 앱(View file)에서 인증서 목록과 내용을 확인할 수 있음
  - 여기서도 바로 인증서를 export 가능

## 인증서 목록을 pem으로 바꾸기
- openssh를 이용하여 p12파일로 부터 인증서를 pem형식으로 가져오기
- 참고 자료 : [Converting a Java Keystore into PEM Format](https://stackoverflow.com/questions/652916/converting-a-java-keystore-into-pem-format)
- 아래와 같이 통채로 pem으로 바꾸기
``` sh
$ openssl pkcs12 -in keystore.p12 -out certificates.pem
Enter Import Password:
MAC verified OK
Enter PEM pass phrase:
```
- 이 중 alias를 보고 본인의 Server Certificate를 찾으면 됨
  - 나머지는 Chain Certificate

## PrivateKey를 꺼내자
- 참고 자료 : [How can I export my private key from a Java Keytool keystore?](https://security.stackexchange.com/questions/3779/how-can-i-export-my-private-key-from-a-java-keytool-keystore)
- p12파일안에 PrivateKey도 저장되어 있음
- 아래와 같이 openssl을 이용해 pem파일로 추출
``` sh
$ openssl pkcs12 -in keystore.p12 -nodes -nocerts -out key.pem
Enter Import Password:
MAC verified OK
```

## 기타 도움되는 자료
- https://www.securesign.kr/guides/SSL-Certificate-Convert-Format

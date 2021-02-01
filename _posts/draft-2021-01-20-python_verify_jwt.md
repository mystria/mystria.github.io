---
layout: post
title:  "Python에서 JWT 검증하기"
date:   2021-01-20 20:00:00 +0900
categories: Python JWT
comments: true
---

# Python에서 JWT 검증하기

# JWT의 활용
[How To Validate a JWT Token](https://medium.com/dataseries/public-claims-and-how-to-validate-a-jwt-1d6c81823826): JWT의 구조와 활용 방법(방법별 장단점)

# PyJWT
[PyJWT의 사용법](https://pyjwt.readthedocs.io/en/stable/usage.html)
RSA로 Asynchronous하게 secret 관리 - JWKS 사용
[Retrieve RSA signing keys from a JWKS endpoint](https://pyjwt.readthedocs.io/en/stable/usage.html#retrieve-rsa-signing-keys-from-a-jwks-endpoint)

## PyJWT에서 RSA(RS256)를 사용하려면?
[Cryptography](https://cryptography.io/en/latest/installation.html)설치 - 해당 package가 env에 존재하면 자동으로 활성화 됨

~~~ python
import jwt
from jwt import PyJWKClient


class TokenVerifier:
    def __init__(self, jwks_url: str):
        # 보안을 위해 jwks_url은 내가 제공해야 함
        self.jwks_url = jwks_url
  
    def verify(self, token: str):
        algorithm = jwt.get_unverified_header(token).get("alg")
        jwks_client = PyJWKClient(self.jwks_url)
        # Env에 cryptography package가 존재해야 RSA를 decode함
        signing_key = jwks_client.get_signing_key_from_jwt(token)
    
        data = jwt.decode(
          token,
          signing_key.key,
          algorithms=[algorithm],
          options={"verify_exp": False},
        )
        return data
~~~
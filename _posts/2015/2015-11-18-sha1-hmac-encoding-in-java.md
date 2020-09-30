---
layout: post
title: SHA1 HMAC encoding in Java
categories: []
tags: []
published: True

---

## sha1 hmac

### HMAC

>MAC that uses a cryptographic hash function in conjunction with a secret key.

mac : Message Authentication Code - 무결성을 검증

따라서 sha1 hmac 란 sha1를 해시 함수로 쓰는 메시지 무결성 검증 코드이다.


## Generating sha1 hmac signature with Java

 - mac 객체 생성    : `Mac mac = Mac.getInstance(해시알고리즘).init(new SecretKeySpec(키값, 해시알고리즘))`;
 - 암호화                 : `byte[] rawHmac = mac.doFinal(암호화할 데이터)`
 - base64 인코딩   : `Base64.encode(rawHmac);`

```java
private static final String HMAC_SHA1_ALGORITHM = "HmacSHA1";

public String calculateHMAC(String data, String key) throws java.security.SignatureException {
        try {
            SecretKeySpec signingKey = new SecretKeySpec(key.getBytes(),HMAC_SHA1_ALGORITHM);
            Mac mac = Mac.getInstance(HMAC_SHA1_ALGORITHM);
            mac.init(signingKey);
            byte[] rawHmac = mac.doFinal(data.getBytes());
            return Base64.encode(rawHmac);
        } catch (Exception e) {
            throw new SignatureException("Failed to generate HMAC : " + e.getMessage());
        }
    }
```

동일한 키로 데이터를 signature로 바꾼 후에 비교해서 무결성을 검증한다.

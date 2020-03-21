---
layout: post
title: "JWT 사용하기"
excerpt: 스프링부트를 이용한 eatgo 프로젝트-day11 (JWT)
---

# JWT (JSON Web Tokens)
  
JWT는 JSON 포맷의 웹 토큰이다. 

access token 발급을 위해 JWT를 사용할 것이다.

JWT는 Header, Payload, Signature로 구성되어 있다.

`Header.Payload.Signature` 의 형식으로 Base64로 인코딩되며 `.`으로 연결된다.

- Header : 타입, 알고리즘
- Payload : 실제 데이터 암호화x, 외부 노출 데이터 담으면 안된다.
- Signature : 위조 변조 되지 않았음을 증명하는 서명

Payload에 담기는 내용을 - Claims라고 한다.

프로젝트에서는 HMAC-SHA256 암호화를 할 것이다.

[https://jwt.io/](https://jwt.io/)에 가보면 jwt를 인코딩/디코딩 할 수있고 개발환경에 맞는 버전을 확인할 수 있다.

star를 가장 많이받은 `io.jsonwebtoken`을 사용할 것이다.

`build.gradle`에 jwt 관련하여 dependency를 추가해주자.

```groovy
implementation 'io.jsonwebtoken:jjwt-api:0.11.1'
runtime 'io.jsonwebtoken:jjwt-impl:0.11.1'
runtime 'io.jsonwebtoken:jjwt-jackson:0.11.1'
```

`JwtUtil`을 만들어 createToken과 getClaims를 만들 것이다.

```java
public class JwtUtil {

    String secret = "abcdefghijklnmopqrstuvwxyz123456"; //secretKey

    public String createToken(Long userId, String name) {
        Key key = Keys.hmacShaKeyFor(secret.getBytes());

        return Jwts.builder()
                .claim("userId", userId)
                .claim("name", name)
                .signWith(key, SignatureAlgorithm.HS256)
                .compact();
    }
}
```

`io.jsonwebtoken.Jwts`를 임포트해주고 claim에 userId와 name을 넣어주었다.  
secretKey와 암호화 알고리즘도 설정해주었다.

test를 돌려보면 아래와 같은 토큰을 얻을 수 있다.

`eyJhbGciOiJIUzI1NiJ9.eyJ1c2VySWQiOjEsIm5hbWUiOiJjb2NvZGV2In0.g7BJHpONXgWRe-YCBurhTLvB_hG30V45I1Z9TW8qhxs`

이제 토큰을 넣어주면 Claims를 얻는 메서드를 구현해보자.
```java
public class JwtUtil {
    //...
    public Claims getClaims(String token) {
        return Jwts.parserBuilder()
                .setSigningKey(key)
                .build()
                .parseClaimsJws(token)
                .getBody();
    }
}
```

Jwt에 대한 parser를 만들어주었다. `getClaims(token)`을 이용하여 넣어주었던 데이터를 받아올 수 있다.

이제 Claims에서 `claims.get("userId", Long.class)`, `claims.get("name", String.class)` 형식으로 id, name을 받아 올 수 있다.
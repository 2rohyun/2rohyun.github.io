---
layout: post
title:  "Spring Security with JWT #1. 배경지식 및 의사결정"
summary: "Spring Security with JWT #1. 배경지식 및 의사결정"
author: 2rohyun
date: '2021-08-05 15:45:00 +0900'
category: spring
thumbnail: /assets/img/posts/jwt.png
keywords: spring security, jwt, session, jjwt, auth0, nimbus
permalink: /blog/spring-jwt-with-security/
usemathjax: false
---

# Spring Security with JWT #1. 배경지식 및 의사결정

## 0. 들어가기 앞서

평소 세션에 기반한 인증과 토큰에 기반한 인증 모두 사용해보고, 구현해 보았지만 Spring Security 의 아키텍처 및 인증과 권한의 구조에 대하여 깊이 이해하고 사용한다는 느낌을 갖지 못하였다. 그러던 중, 회사의 새로운 프로젝트와 8월에 출전하는 공개SW 개발자 대회에서 JWT 를 이용한 인증 및 권한 시나리오가 필요하게 되었고, 이에 따라 전체적인 배경지식부터 의사결정, 그리고 설계 및 구현에 대하여 정리하고 문서화하고자 한다. 또한 이번 포스트 시리즈에서는 Spring Security 및 JWT 구현, Redis 를 이용한 Refresh token 구현 및 저장에 초점을 둘 것이기에, 인증 및 권한, 세션 기반 인증 및 토큰 기반 인증, Bearer token 등의 선수 지식에 대해서는 간단하게만 정리하려고 한다.

## 1. 인증과 권한

- 인증(Authentication) : 사이트에 접근하는 사람이 누구인지 시스템이 알아야 한다. 익명사용자(anonymous user)를 허용하는 경우도 있지만, 특정 리소스에 접근하거나 개인화된 사용성을 보장 받기 위해서는 반드시 로그인하는 과정이 필요하다. 로그인은 보통 username / password 를 입력하고 로그인하는 경우와 sns 사이트를 통해 인증을 대리하는 경우가 있다.

    - Username / password 인증
        - 세션 기반 인증
        - 토큰 기반 인증 ( STATELESS )
    - SNS 로그인 ( 인증 위임 )


- 권한(Authorization) : 사용자가 누구인지 알았다면 사이트 관리자 혹은 시스템은 로그인한 사용자가 어떤 일을 할 수 있는지 권한을 설정한다. 권한은 특정 페이지에 접근하거나 특정 리소스에 접근할 수 있는 권한여부를 판단하는데 사용된다. 개발자는 권한이 있는 사용자에게만 페이지나 리소스 접근을 허용하도록 코딩해야 하는데, 이런 코드를 쉽게 작성할 수 있도록 프레임워크를 제공하는 것이 스프링 시큐리티 프레임워크(Spring Security Framework) 이다.

> 쉽게 예를 들어 설명하자면, 인증이란 건물 내에 출입할 수 있는 지의 여부를 판단하는 것이고, 권한이란 건물 내의 어떠한 층이나 방에 출입할 수 있는 지의 여부라고 생각하면 편할 것이다.

## 2. Session vs. Token

- 세션 기반 인증
    - 유저가 로그인을 하면 Session 이 서버 단의 메모리에 저장된다. 이 때 Session 을 식별하기 위한 Session ID 를 기준으로 정보를 저장한다.
    - 브라우저에 쿠키로 Session ID 가 저장된다.
    - 쿠키에 정보가 담겨있기 때문에 브라우저는 해당 사이트에 대한 모든 Request 에 Session ID 를 쿠키에 담아 전송한다.
    - 서버는 클라이언트가 보낸 Session ID 와 서버 메모리로 관리하고 있는 Session ID 를 비교하여 Verification 을 수행한다.


- 세션 기반 인증의 장점
    - 상대적으로 안전하다. 서버측에서 관리하기 때문에 클라이언트 변조에 영향받거나 데이터 손상 우려가 없다.
    - 직접 경험해본 바로, Spring Security 에서 토큰 기반의 인증 방식보다 구현이 단순하고 명료하며, RememberMe Token 과 같은 다양한 기능을 편하게 사용할 수 있다.


- 세션 기반 인증의 단점
    - 서버를 여러대 둘 경우 (scale out) 또는 같은 사용자가 서로 다른 도메인의 데이터를 요청할 경우, (SSO) 에는 세션을 유지하기 위한 비용이 매우 커지게 된다.
    - 서버의 메모리에 세션을 저장해야하므로 서버에 부담을 주게된다.


- 토큰 기반 인증
    - 유저가 로그인을 하면 Token 을 발급한다.
    - 클라이언트는 발급된 Token 을 저장한다. ( Local storage, Cookie 등 )
    - 클라이언트는 요청 시 저장된 Token 을 Header Authorization 필드에 포함시켜 보낸다. 
    - 서버는 매 요청시 클라이언트로부터 전달받은 Header 의 Token 정보를 Verification 한 뒤, 해당 유저에 권한을 인가한다.


- 토큰 기반 인증의 장점
    - 클라이언트에 저장되기 때문에 서버의 메모리에 부담이 되지않으며 Scale out 에 대한 추가적인 소요가 없다.
    - 멀티 디바이스 환경에 적합하다.


- 토큰 기반 인증의 단점
    - 이미 발급된 JWT에 대해서는 돌이킬 수 없다. 따라서 탈취되어도 토큰 유효기간동안 계속해서 사용할 수 있다. ( 보완책 ) Refresh Token
    - Refresh Token 발급에 따른 서버의 메모리 소모가 발생한다.

## 3. JWT 

- JWT(Json Web Token) 란? Json 포맷을 이용하여 사용자에 대한 속성을 저장하는 Claim 기반의 Web Token이다. JWT는 토큰 자체를 정보로 사용하는 Self-Contained 방식으로 정보를 안전하게 전달한다.

- 클라이언트에서 JWT를 HTTP 헤더의 Authorization 필드에 담아 요청을 보내면 서버는 유효한 JWT 인지를 검사한다. 또한 로그아웃을 할 경우 로컬 스토리지에 저장된 JWT 데이터를 제거한다. 또한 실제 서비스에서는 로그아웃 요청을 한 토큰을 서버 단에 blacklist 로 저장해두어 해당 토큰의 접근을 막아야 한다.

- JWT 의 구조
    - Header
        - typ: 토큰의 타입을 지정 ex) JWT
        - alg: 알고리즘 방식을 지정하며, 서명(Signature) 및 토큰 검증에 사용 ex) HS256(SHA256) 또는 RSA


    - Payload
        - 토큰의 페이로드에는 토큰에서 사용할 정보의 조각들인 클레임(Claim)이 담겨 있다. 
        - iss: 토큰 발급자(issuer)
        - sub: 토큰 제목(subject)
        - aud: 토큰 대상자(audience)
        - exp: 토큰 만료 시간(expiration), NumericDate 형식으로 되어 있어야 함 ex) 1480849147370
        - nbf: 토큰 활성 날짜(not before), 이 날이 지나기 전의 토큰은 활성화되지 않음
        - iat: 토큰 발급 시간(issued at), 토큰 발급 이후의 경과 시간을 알 수 있음
        - jti: JWT 토큰 식별자(JWT ID), 중복 방지를 위해 사용하며, 일회용 토큰(Access Token) 등에 사용


    - Signature
        - 서명(Signature)은 토큰을 인코딩하거나 유효성 검증을 할 때 사용하는 고유한 암호화 코드이다. 서명(Signature)은 위에서 만든 헤더(Header)와 페이로드(Payload)의 값을 각각 BASE64로 인코딩하고, 인코딩한 값을 비밀 키를 이용해 헤더(Header)에서 정의한 알고리즘으로 해싱을 하고, 이 값을 다시 BASE64로 인코딩하여 생성한다.

## 4. JWT([RFC7519](https://datatracker.ietf.org/doc/html/rfc7519)) vs. Bearer Token([RFC6750](https://datatracker.ietf.org/doc/html/rfc6750))

- Bearer Token 이란? RFC6750 의 Terminology 에 따르면, 

> A security token with the property that any party in possession of the token (a "bearer") can use the token in any way that any other party in possession of it can. Using a bearer token does not require a bearer to prove possession of cryptographic key material (proof-of-possession).

- JWT는 서명 및 암호화할 수 있는 JSON 데이터 페이로드를 포함하는 토큰의 인코딩 표준이다.

- JWT는 여러 가지 용도로 사용될 수 있습니다. 그 중 하나는 Bearer Token 이다. 즉, 사용자가 가지고 있기 때문에(당신이 "배어러"가 됨으로써) 어떤 서비스에 액세스할 수 있는 정보를 제공할 수 있다.

- 보유자 토큰은 여러 가지 방법으로 HTTP 요청에 포함될 수 있으며, 그 중 (아마도 선호되는) 하나는 권한 부여 헤더이다. 그러나 요청 매개 변수, 쿠키 또는 요청 본문에 넣을 수도 있다. 대부분 액세스하려는 서버와 사용자 간에 발생한다.

- 단순하게 말하면, JWT는 클레임을 인코딩하고 확인할 수 있는 편리한 방법! 베어러 토큰은 권한 부여에 사용되는 문자열일 뿐이다!

## 5. JWT implementation Libraries for Java

- 라이브러리 종류
    - com.auth0 / java-jwt 
    - org.bitbucket.b_c / jose4j
    - com.nimbusds / nimbus-jose-jwt 
    - io.jsonwebtoken / jjwt 
    - com.inversoft / prime-jwt 
    - io.vertx / vertx-auth-jwt 

- 위와 같이 JWT 를 구현할 수 있는 다양한 라이브러리들이 존재한다. 나는 jjwt 와 com.auth0 를 이용하여 JWT 를 구현해보았고, jose4j 와 nimbus-jose-jwt 도 많이 사용되어지고 레퍼런스 또한 많지만 새로운 구현 방식을 배우는 소요가 든다고 판단하여 io.jsonwebtoken 이나 com.auth0 을 고려하였다.

- [jjwt github link](https://github.com/jwtk/jjwt) 

- [auth0 github link](https://github.com/auth0/java-jwt)

- 두 레포지토리의 스타 수와 컨트리뷰션을 보았을 떄, 괄목할만한 차이를 보이지 않았고 둘 다 많은 레퍼런스를 보유하고 있었다. 오히려 다양한 블로그를 참고했을 때, 한국에서는 jjwt 를 훨씬 더 많이 사용하고 있는 것으로 보여졌다. 하지만 이 것은 의사 결정의 중요한 요인은 되지 못한다고 생각하였고, 두 방식으로 모두 JWT 를 구현해본 바, 나에게 조금이라도 더 직관적이면서 쉽고 간편하게 다가오는 auth0 에서 제공해주는 라이브러리를 사용하기로 하였다.


### REFERENCE

https://datatracker.ietf.org/doc/html/rfc7519

https://datatracker.ietf.org/doc/html/rfc6750

https://stackoverflow.com/questions/40375508/whats-the-difference-between-jwts-and-bearer-token

https://mangkyu.tistory.com/56 [MangKyu's Diary]

https://jins-dev.tistory.com/entry/Session-기반-인증과-Token-기반-인증 [Jins' Dev Inside]

https://tansfil.tistory.com/58



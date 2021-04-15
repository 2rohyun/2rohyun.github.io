---
layout: post
title:  "spring security authentication 메커니즘"
summary: "spring security 인증"
author: 2rohyun
date: '2021-04-15 12:42:23 +0900'
category: spring
thumbnail: /assets/img/posts/springboot.png
keywords: spring, spring security, authentication
permalink: /blog/spring-security-authentication/
usemathjax: false
---
# Spring Security Authentication 메커니즘
## 인증 (Authentication )
![photo1](https://gitlab.com/jongwons.choi/spring-boot-security-lecture/-/raw/master/images/fig-6-Authentication.png)

- Authentication 은 인증을 하기 위한 정보와 인증을 받기 위한 정보가 하나의 객체에 동시에 들어있다. 왜냐하면 AuthenticationProvider ( 인증 제공자 ) 는 어떤 인증에 대하여 허가를 내줄 것인지 판단하기 위해서는 직접 입력된 인증을 보고 허가된 인증을 내주는 방식이기 때문이다.  따라서 authenticate( ) 메소드에서 Authentication 객체를 입력과 동시에 출력으로 사용한다. 

- Authentication 객체를 구성하는 요소들
— Credentials : 인증을 받기 위해 필요한 정보, 비번등 (input)
— Principal : 인증된 결과. 인증 대상 (output)
— Details : 기타 정보, 인증에 관여된 된 주변 정보들
— Authorities : 권한 정보들

- Authentication 을 구현한 객체를 “ 인증 토큰 “ 이라고 부르기도 한다.
- Authentication 객체는 SecurityContextHolder 를 통해 세션이 있건 없건 언제든 접근할 수 있도록 필터체인에서 보장해 준다.

## 인증 제공자 (Authentication Provider)
![photo2](https://gitlab.com/jongwons.choi/spring-boot-security-lecture/-/raw/master/images/fig-8-AuthenticationManager.png)

- 인증 제공자(AuthenticationProvider)는 기본적으로 Authentication 을 받아서 인증을 하고 인증된 결과를 다시 Authentication 객체로 전달한다.
- 인증 제공자는 어떤 인증에 대해서 도장을 찍어줄지 AuthenticationManager 에게 알려줘야 하기 때문에 support() 라는 메소드를 제공한다. 인증 대상과 방식이 다양할 수 있기 때문에 인증 제공자도 여러개 올 수 있다.

## 인증 관리자 (Authentication Manager)
![photo3](https://gitlab.com/jongwons.choi/spring-boot-security-lecture/-/raw/master/images/fig-8-AuthenticationManager.png)
- AuthenticationProvider 들을 관리하는 인터페이스가 AuthenticationManager (인증 관리자)이고, 이 AuthenticationManager 를 구현한 객체가 ProviderManager 입니다.
- ProviderManager 도 여러개가 존재할 수 있다.
- 개발자가 직접 AuthenticationManager를 정의해서 제공하지 않는다면, AuthenticationManager 를 만드는 AuthenticationManagerFactoryBean 에서 DaoAuthenticationProvider 를 기본 AuthenticationProvider 로 등록한 AuthenticationManager 를 만든다.
- * DaoAuthenticationProvider 는 반드시 1개의 UserDetailsService 를 발견할 수 있어야 한다. 만약 없으면 InmemoryUserDetailsManager 에 [username=user, password=(서버가 생성한 패스워드)]인 사용자가 등록되어 제공된다.

## REFERENCE
https://gitlab.com/jongwons.choi/spring-boot-security-lecture/
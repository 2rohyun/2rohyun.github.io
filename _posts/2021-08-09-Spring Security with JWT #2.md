---
layout: post
title:  "Spring Security with JWT #2. Spring Security Architecture"
summary: "Spring Security with JWT #2. Spring Security Architecture"
author: 2rohyun
date: '2021-08-09 18:45:00 +0900'
category: spring-security
thumbnail: /assets/img/posts/jwt.png
keywords: spring security, jwt, architecture
permalink: /blog/spring-jwt-with-security-two/
usemathjax: false
---

## 0. 들어가기 앞서

본격적으로 auth0.com 에서 제공해주는 jwt implementation libarary 를 사용하여 jwt 를 구현하기 전에, 인프런의 백기선님의 강의를 참고하여 Spring Security Architecture 에 대해 divide-and-conquer 방식으로 이해를 돕고자 한다.

## 1. SecurityContextHolder 와 Authentication

![contextholder](/assets/img/posts/contextholder.png){: width="33%" height="33%"}

- SecurityContextHolder : SecurityContext 제공, 기본적으로 ThreadLocal 을 사용한다.

> ThreadLocal 이란? Java.lang 패키지에서 제공하는 쓰레드 범위 변수, 즉, 쓰레드 수즌의 데이터 저장소이다. 같은 쓰레드 내에서만 공유되어지고, 따라서 같은 쓰레드라면 해당 데이터를 메소드 매개변수로 넘겨줄 필요가 없다.

- SecurityContext : Authentication 제공
- Authentication : Principal 과 GrantAuthority 제공
- Principal 
    - `누구`에 해당하는 정보
    - UserDetailsService 에서 리턴한 객체 ( UserDetails )
- GrantAuthority
    - "ROLE_USER", "ROLE_ADMIN" 등 Principal 이 가지고 있는 권한을 나타낸다.
    - 인증 이후, 인가 및 권한 확인할 때 이 정보를 참조한다.
- UserDetails : 애플리케이션이 가지고 있는 유저 정보와 스프링 시큐리티가 사용하는 Authentication 객체 사이의 어댑터
- UserDetailsService : 유저 정보를 UserDetails 타입으로 가져오는 DAO 인터페이스

```java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
Object principal = authentication.getPrincipal();
Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
Object credentials = authentication.getCredentials();
boolean authenticated = authentication.isAuthenticated();
```

> Async 를 명시적으로 사용하지 않는 한, 하나의 Request 당 하나의 thread 가 사용되고, SecurityContextHolder 는 한 쓰레드 내에서 공유하는 저장소인 ThreadLocal 를 사용하므로 별도의 파라미터를 지정해주지 않아도 thread 내에서 요청한 User 에 대한 Authentication 을 가져올 수 있게된다.

## 2. AuthenticationManager 와 Authentication

- 스프링 시큐리티에서 인증(Authentication) 은 AuthenticationManager 가 수행한다.

```java
Authentication authenticate(Authentication authentication) throws AuthenticationException;
```

- `인자로 받은 Authentication 이 유효한 인증인지 확인하고 Authentication 객체를 리턴한다.`
- 인증을 확인하는 과정에서 비활성 계정, 잘못된 비밀번호, 잠긴 계정 등의 에러를 던질 수 있다.

- 인자로 받은 Authentication
    - 사용자가 입력한 인증에 필요한 정보(username, password) 로 만든 객체 (폼 인증인 경우)
    - Authentication
        - Principal : "dohyun"
        - Credentials : "123"

- 유효한 인증인지 확인
    - 사용자가 입력한 password 가 UserDetailsService 를 통해 읽어온 UserDetails 객체에 들어있는 password 와 일치하는지 확인
    - 해당 사용자 계정이 잠겨 있진 않은지, 비활성 계정은 아닌지 등 확인

- Authentication 객체를 리턴
    - Authentication
        - Principal : UserDetailsService 에서 리턴한 객체 (User)
        - Credentials : null
        - GrantedAuthorities

- AuthenticationManager 인터페이스에는 authenticate() 메소드만 존재하고 인터페이스에 대한 구현체로 보통 ProviderManager 를 사용한다. 

## 3. Authentication 과 SecurityContextHolder

- AuthenticationManager 가 인증을 마친 뒤 리턴 받은 Authentication 객체의 행방은?
- UsernamePasswordAuthenticationFilter
    - 폼 인증을 처리하는 시큐리티 필터
    - 인증된 Authentication 객체를 SecurityContextHolder 에 넣어주는 필터
    - SecurityContextHolder.getContext().setAuthentication(authentication)

- SecurityContextPersistenceFilter
    - SecurityContext 를 HTTP session 에 캐시(기본 전략)하여 여러 요청에서 Authentication 을 공유할 수 있게해주는 필터
    - SecurityContextRepository 를 교체하여 세션을 HTTP session 이 아닌 다른 곳에 저장하는 것도 가능하다.

> JWT 의 경우 STATEFUL 한 session 을 사용하지 않고 STATELESS 방식으로 구현하기 때문에 매 요청마다 인증을 해야한다. 따라서 나는 OncePerRequestFilter 클래스를 상속받아 커스터마이징하여 구현할 것이다.

## 4. 스프링 시큐리티 Filter 와 FilterChainProxy

- 스프링 시큐리티가 제공하는 필터들
    - WebAsyncManagerIntergrationFilter
    - SecurityContextPersistenceFilter
    - HeaderWriterFilter
    - CsrfFilter
    - LogoutFilter
    - UsernamePasswordAuthenticationFilter
    - DefaultLoginPageGeneratingFilter
    - DefaultLogoutPageGeneratingFilter
    - BasicAuthenticationFilter
    - RequestCacheAwareFilter
    - SecurityContextHolderAwareRequestFilter
    - AnonymousAuthenticationFilter
    - SessionManagementFilter
    - ExceptionTranslationFilter
    - FilterSecurityInterceptor

- 이 모든 필터는 `FilterChainProxy` 가 호출한다.

![filterchainproxy](/assets/img/posts/filterchainproxy.png){: width="33%" height="33%"}

## 5. DelegatingFilterProxy 와 FilterChainProxy

- DelegatingFilterProxy
    - 일반적인 서블릿 필터
    - 서블릿 필터 처리를 스프링에 들어있는 빈으로 위임하고 싶을 때 사용하는 서블릿 필터
    - 타겟 빈 이름을 설정한다.
    - 스프링 부트를 사용할 때는 자동으로 등록된다.(SecurityFilterAutoConfiguration)

- FilterChainProxy
    - 보통 "springSecurityFilterChain" 이라는 이름의 빈으로 등록된다.
 
![delegating](/assets/img/posts/delegating.png){: width="50%" height="50%"}

## 6. AccessDecisionManager 와 FilterSecurityInterceptor

- Access Control 결정을 내리는 인터페이스로, 구현체 3가지를 기본으로 제공한다.
    - AffirmativeBased : 여러 Voter 중 한 명이라도 허용하면 허용, 기본 전략
    - ConsensusBased : 다수결
    - UnanimousBased : 만장일치

- AccessDecisionVoter
    - 해당 Authentication 이 특정한 Object 에 접근할 때 필요한 ConfigAttributes 를 만족하는지 확인한다.
    - WebExpressionVoter : 웹 시큐리티에서 사용하는 기본 구현체, ROLE_XXXX 가 매치하는지 확인한다.

- FilterSecurityInterceptor : AccessDecisionManager 를 사용하여 Access Control 또는 예외 처리를 하는 필터. 대부분의 경우 FilterChainProxy 의 제일 마지막 필터로 들어있다.

## 7. 스프링 시큐리티 아키텍처 정리

![securityarchitecture](/assets/img/posts/securityarchitecture.png){: width="66%" height="66%"}

### REFERENCE

[백기선 님 인프런 강의](https://www.inflearn.com/course/%EB%B0%B1%EA%B8%B0%EC%84%A0-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0/dashboard)

  

---
layout: post
title:  "spring security 에 대하여"
summary: "spring security"
author: 2rohyun
date: '2021-04-15 12:42:23 +0900'
category: spring-security
thumbnail: /assets/img/posts/springboot.png
keywords: spring, spring security
permalink: /blog/about-spring-security/
usemathjax: false
---
# Spring Security 에 대하여

## security가 필요한 이유

웹 사이트는 그 사이트 자체의 리소스가 있고, 그 웹 사이트를 이용하는 유저들의 개인 정보를 가지고 있다. 악의적으로 리소스나 개인정보를 탈취하려는 행동을 막을 방법이 필요하게 되고 그러한 것들을 서포트 해주는 것이 spring security 이다.
Security 는 크게 두 가지로 나뉜다. 
- 인증 ( Authentication )
- 권한 ( Authorization )

## 인증 ( Authentication )

사이트에 접근하는 사람이 누구인지 시스템이 알아야 합니다. 익명사용자(anonymous user)를 허용하는 경우도 있지만, 특정 리소스에 접근하거나 개인화된 사용성을 보장 받기 위해서는 반드시 로그인하는 과정이 필요하다. 
로그인은 보통 username / password 를 입력하고 로그인하는 경우와 sns 사이트를 통해 인증을 대리하는 경우가 있다.
- user name / password 인증
— Session 관리
— token 관리 ( sessionless ) —> 최근에 사용하는 방법 
- Sns 로그인 (소셜 로그인) : 인증 위임

## 권한 ( Authorization )

사용자가 누구인지 알았다면 사이트 관리자 혹은 시스템은 로그인한 사용자가 어떤 일을 할 수 있는지 권한을 설정한다. 
권한은 특정 페이지에 접근하거나 특정 리소스에 접근할 수 있는 권한여부를 판단하는데 사용됩니다. 개발자는 권한이 있는 사용자에게만 페이지나 리소스 접근을 허용하도록 코딩해야 하는데, 이런 코드를 쉽게 작성할 수 있도록 프레임워크를 제공하는 것이 스프링 시큐리티 프레임워크(Spring Security Framework) 이다.
- Secured : deprecated —> annotation check
- PrePostAuthorize —> annotation check
어노테이션으로 권한을 설정하기 복잡해지는 로직이 된다면
- AOP를 이용하여 권한을 check

## 서블릿 컨테이너

톰켓과 같은 웹 애플리케이션을 서블릿 컨테이너라고 부르는데, 이런 웹 애플리케이션(J2EE Application)은 기본적으로 필터와 서블릿으로 구성

![photo](/assets/img/posts/servlet-container.png){: width="50%" height="50%"}

 - 필터는 체인처럼 엮여있기 때문에 필터 체인이라고도 불리는데, 모든 request 는 이 필터 체인을 반드시 거쳐야만 서블릿 서비스에 도착하게 된다.

## Security Filter

![photo2](/assets/img/posts/security-filter.png){: width="50%" height="50%"}

- 스프링 시큐리티는 DelegatingFilterProxy 라는 필터를 만들어 메인 필터체인에 끼워넣고, 그 아래 다시 SecurityFilterChain 그룹을 등록
- 이 필터체인은 반드시 한개 이상이고, url 패턴에 따라 적용되는 필터체인을 다르게 할 수 있다. 본래의 메인 필터를 반드시 통과해야만 서블릿에 들어갈 수 있는 단점을 보완하기 위해서 필터체인 Proxy 를 두었다.
- web resource 의 경우 패턴을 따르더라도 필터를 무시(ignore)하고 통과시켜주기도 한다.

## Filter의 종류

*HeaderWriterFilter* : Http 해더를 검사한다. 써야 할 건 잘 써있는지, 필요한 해더를 더해줘야 할 건 없는가?

*CorsFilter* : 허가된 사이트나 클라이언트의 요청인가?

*CsrfFilter* : 내가 내보낸 리소스에서 올라온 요청인가?

*LogoutFilter* : 지금 로그아웃하겠다고 하는건가?

*UsernamePasswordAuthenticationFilter* : username / password 로 로그인을 하려고 하는가? 만약 로그인이면 여기서 처리하고 가야 할 페이지로 보내 줄께.

*ConcurrentSessionFilter* : 여거저기서 로그인 하는걸 허용할 것인가?

*BearerTokenAuthenticationFilter* : Authorization 해더에 Bearer 토큰이 오면 인증 처리 해줄께.

*BasicAuthenticationFilter* : Authorization 해더에 Basic 토큰을 주면 검사해서 인증처리 해줄께.

*RequestCacheAwareFilter* : 방금 요청한 request 이력이 다음에 필요할 수 있으니 캐시에 담아놓을께.

*SecurityContextHolderAwareRequestFilter* : 보안 관련 Servlet 3 스펙을 지원하기 위한 필터라고 한다.(?)

*RememberMeAuthenticationFilter* : 아직 Authentication 인증이 안된 경우라면 RememberMe 쿠키를 검사해서 인증 처리해줄께

*AnonymousAuthenticationFilter* : 아직도 인증이 안되었으면 너는 Anonymous 사용자야

*SessionManagementFilter* : 서버에서 지정한 세션정책을 검사할께.

*ExcpetionTranslationFilter* : 나 이후에 인증이나 권한 예외가 발생하면 내가 잡아서 처리해 줄께.

*FilterSecurityInterceptor* : 여기까지 살아서 왔다면 인증이 있다는 거니, 니가 들어가려고 하는 request 에 들어갈 자격이 있는지 그리고 리턴한 결과를 너에게 보내줘도 되는건지 마지막으로 내가 점검해 줄께.

- 필터는 넣거나 뺄 수 있고 순서를 조절할 수 있다. (이때 필터의 순서가 매우 critical 할 수 있기 때문에 기본 필터들은 그 순서가 어느정도 정해져 있다.)

## REFERENCE
https://gitlab.com/jongwons.choi/spring-boot-security-lecture/

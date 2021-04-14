---
layout: post
title:  "Spring boot test 에 대하여"
summary: "spring boot test 에 대한 기본적 개념 정리"
author: 2rohyun
date: '2021-04-14 10:47:23 +0900'
category: spring
thumbnail: /assets/img/posts/springboot.png
keywords: spring, spring boot, test, spring boot test, mock, unit, junit
permalink: /blog/spring-boot-test/
usemathjax: false
---
# Spring boot test 

## 0. 들어가기 앞서
회사에서 새로운 프로젝트의 서버 개발을 맡아 로그인, 회원가입을 비롯한 다양한 기능을 구현하던 중, Front-end 와의 통신을 통한 브라우저 상의 테스트만으로는 다양한 시나리오에서 발생할 수 있는 예기치 못한 상황에 대한 핸들링이 부족하다고 생각하였고, 테스트 코드를 작성하고 커버리지를 측정하여 수치화된 신뢰성 있는 코드를 작성하고, 궁극적으로 기능에 대한 불확실성을 감소시키고 싶었다. 기본적인 Repository Test 와 MockMvc 를 통한 테스트는 경험해보았지만, 체계화된 테스트를 위하여 Spring boot test 전반적인 부분에 대하여 공부하고 문서화하고자 한다.

## spring-boot-startet-test 에 포함되어있는 테스트 라이브러리
1. Junit5 ( Junit 4와의 하위 호환성 유지를 위한 빈티지 엔진 포함 ) : Java application 을 위한 표준 단위 테스트
2. AssertJ : A fluent assertion library
3. Hamcrest : matcher 객체의 라이브러리
4. Mockito : Java mocking framework
5. JSONassert : An assertion libarary for JSON
6. JsonPath : XPath for JSON
7. Spring Test & Spring Boot Test : Spring boot application 을 위한 유틸리티와 통합 테스트 지원

## Spring boot 통합 테스트
 - 애플리케이션의 설정, 모든 Bean 을 로드하기 때문에 운영환경과 가장 비슷한 테스트 
 - 전체적인 Flow 를 테스트할 수 있다.

### [1]. @SpringBootTest
 - 스프링 부트 애플리케이션 테스트테 필요한 거의 모든 의존성을 제공해준다.
 - Junit4 를 사용하는 경우 - @RunWith(Springrunner.class) 와 함께 사용해야한다.
 - Junit5 를 사용하는 경우 - 해당 어노테이션을 명시할 필요 없다.

 1. properties : {"key = value"} 형식으로 직접 프로퍼티를 추가할 수 있다.
 ```java
 @SpringBootTest( 
     properties = { "propertyTest.value=propertyTest", 
                    "testValue=test" })
 ```

2. webEnvironment : 웹 테스트 환경을 선택할 수 있다.

* Mock
 - 실제 객체를 만들기에 비용과 시간이 많이 들거나 의존성이 연관되어 있어 제대로 구현하기 힘들 경우 가짜 객체를 만들어서 사용한다.
 - 내장 서블릿 컨테이너가 아닌 Mock 서블릿을 제공한다.
 - @AutoConfigureMockMvc : Mock 테스트 시 필요한 의존성을 제공해준다. 
 - MockMvc : 브라우저에서 요청과 응답을 의미하는 객체로서 Controller 테스트에 사용한다.

* RANDOM_PORT
 - EmbeddedWebApplicationContext 를 로드하며 실제 서블릿 환경 구성 (임의의 port를 listen)

* DEFINED_PORT 
 - 실제 서블릿 환경 구성 (프로퍼티에서 지정한 포트를 listen)

* NONE
 - 기본적인 ApplicationContext 를 로드한다.

> Application Context 란?
 - Web Application 최상단에 위치
 - BeanFactory 를 상속받고 있는 Context
 - Bean 에 대한 IoC Container
 - 서블릿 설정과 관계 없는 설정을 한다.(@Service, @Repository, @Configuration, @Component ..)
 - 서로 다른 여러 서블릿에서 공통으로 공유해서 사용할 수 있는 Bean 선언
 - Application Context 에 정의된 Bean 은 서블릿 Context 에 정의된 Bean 사용 불가.

* TestRestTemplate
 - webEnvironment 설정에 맞게 자동으로 빈이 생성되며, RestTemplate 를 처리하기 위해 사용된다.
 - Spring 4.x 이후로 부터 지원하는 Spring HTTP 통신 템플릿
 - ResponseEntity 와 Server to Server 통신을 하는 데에 자주 쓰인다.
 - Header, Content-Type 등을 설정하여 외부 API 를 호출하는 데에 쓰인다.

> ResponseEntity 란?
 - 응답 처리 시 값 뿐만 아니라 상태 코드, 응답 메세지 등을 포함하여 리턴할 수 있는 객체이다. HttpEntity 를 상속받기 때문에 HttpHeader 와 body 를 가질 수 있다.

### [2]. @MockBean
 - Mock 객체를 빈으로써 등록할 수 있다.
 - ApplicationContext 의 Mock 객체를 빈으로 등록한다. 만약 @MockBean 으로 선언된 객체와 같은 이름과 타입이 이미 빈으로 등록되어 있다면 해당 빈은 선언한 @MockBean 으로 대체된다.

### [3]. @Transactional
 - 테스트 완료 후 자동 roll back 처리를 한다. (@Test 와 함께 사용)
 - RANDOM_PORT, DEFINED_PORT 를 사용할 경우 실제 테스트 서버는 별도의 스레드에서 테스트를 수행하기 때문에 트랜잭션이 롤백되지 않는다.

## Spring boot 단위 테스트

### [1]. @WebMvcTest
 - MVC 를 위한 테스트, 컨트롤러가 예상대로 동작하는 지 테스트에 사용된다.
 - MockBean, MockMvc 를 자동으로 구성하여 테스트할 수 있다.
 - Spring Security 를 지원한다.
 - 예시 : `@WebMvcTest(TestController.class)`
 - 통합 테스트보다 빠르지만 모든 테스트를 Mock 기반으로 하기 때문에 실제 환경에서는 동작하지 않을 수 있다.

### [2]. @DataJpaTest
 - JPA 관련 설정만 로드하며, 데이터에 대한 CRUD 테스트가 가능하다.
 - @Entity 클래스를 스캔하여 스프링 데이터 JPA 저장소를 구성한다. (다른 컴포넌트를 스캔하지 않는다.)
 - @Transactional 을 포함하고 있다.
 - 기본적으로 in-memory embedded database 에 대한 테스트를 진행하므로 real database 를 사용하고자 하는 경우 @AutoConfigureTestDatabase 를 사용하면 된다.
 - @AutoConfigureTestDatabase 의 default 값은 any 이므로 `@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)` 를 사용하여 실제 디비를 사용할 수 있다.

## 마치며
당장 나에게 필요한 부분만 정리해놓은 것이다. 테스트에 관한 추가적인 부분 (WebFluxTest, RestClientTest, JsonTest ..) 을 알고 싶다면 [공식문서](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-testing) 를 참고할 것이다.


## REFERENCE
https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-test-scope-dependencies
https://goddaehee.tistory.com/212?category=367461
https://velog.io/@swchoi0329/Spring-Boot-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EC%BD%94%EB%93%9C-%EC%9E%91%EC%84%B1
https://velog.io/@jbb9229/springioccontainer
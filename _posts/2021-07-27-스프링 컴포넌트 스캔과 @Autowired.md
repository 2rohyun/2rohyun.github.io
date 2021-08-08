---
layout: post
title:  "스프링 컴포넌트 스캔과 @Autowired"
summary: "스프링 컴포넌트 스캔과 @Autowired"
author: 2rohyun
date: '2021-07-27 16:45:00 +0900'
category: spring
thumbnail: /assets/img/posts/springboot.png
keywords: spring, di, autowired, component scan
permalink: /blog/spring-autowired-component/
usemathjax: false
---

# 스프링 컴포넌트 스캔과 @Autowired

- 앞선 포스트에서 스프링 빈을 등록할 때는 자바 코드의 @Bean 을 통해서 설정 정보에 직접 등록할 스프링 빈을 나열했다.
- 등록해야할 스프링 빈이 수십, 수백개가 되면 일일이 등록하기도 귀찮고, 설정 정보도 커지고, 누락하는 문제도 발생한다. 
- 그래서 스프링은 설정 정보가 없어도 자동으로 스프링 빈을 등록하는 컴포넌트 스캔이라는 기능을 제공한다.
- 또 의존관계도 자동으로 주입하는 @Autowired 라는 기능도 제공한다.

```java
@Configuration
@ComponentScan
public class AutoAppConfig {

}
```

- 앞선 포스트들에서 수동으로 빈을 등록할 때에는 빈들간의 의존 관계를 수동으로 맺어주었다. 그런데 @ComponentScan 을 이용해 자동으로 빈을 등록하면 의존 관계 설정을을 어떻게 하지?
- 그래서 @Autowired 가 필요하다. @Autowired 는 의존 관계를 자동으로 주입해준다.

![composcan](/assets/img/posts/composcan.png){: width="50%" height="50%"}

- 이때 스프링 빈의 기본 이름은 클래스 명을 사용하되 맨 앞글자만 소문자를 사용한다.
    - 빈 이름 기본 전략 : MemberServiceImpl 클래스 -> memberServiceImpl
    - 빈 이름 직접 지정 : 만약 스프링 빈의 이름을 직접 지정하고 싶으면 `Component("memvberService2")` 

![composcandi](/assets/img/posts/composcandi.png){: width="50%" height="50%"}

- 생성자에 @Autowired 를 지정하면, 스프링 컨테이너가 자동으로 해당 스프링 빈을 찾아서 주입한다.
- 이때 기본 조회 전략은 타입이 같은 빈을 찾아서 주입한다.
    - getBean(MemberRepository.class) 와 동일하다고 이해하면 된다.

## 탐색 위치와 기본 스캔 대상

```java
@ComponentScan(
    basePackages = "hello.core.member",
    basePackageClasses = AutoAppConfig.class,
    excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Configuration.class)
)
```

- basePackages : 탐색할 패키지의 시작 위치를 지정한다. 이 패키지를 포함해서 하위 패키지를 모두 탐색한다.
    - basePackages = {"hello.core", "hello.service"} 이렇게 여러 시작 위치를 지정할 수도 있다.
    - basePackageClasses : 지정한 클래스의 패키지를 탐색 시작 위로 지정한다.
    - default : @ComponentScan 을 명시한 패키지부터 하위 패키지를 모두 스캔한다.

> 권장하는 방법 : 개인적으로 즐겨 사용하는 방법은 패키지 위치를 지정하지 않고, 설정 정보 클래스 위치를 프로젝트 최상단에 두는 것이다. 최근 스프링 부트도 이 방법을 기본으로 제공한다.

> 참고 : 사실 애노테이션에는 상속관계라는 것이 없다. 그래서 애노테이션이 특정 애노테이션을 들고 있는 것을 인식할 수 있는 것은 자바 언어가 지원하는 기능은 아니고, 스프링이 지원하는 기능이다.

- 컴포넌트 스캔의 용도 뿐만 아니라 다음 애노테이션이 있으면 스프링은 부가 기능을 수행한다.
    - @Controller : 스프링 MVC 컨트롤러로 인식
    - @Repository : 스프링 데이터 접근 계층으로 인식하고, 데이터 계층의 예외를 스프링 예외로 변환해준다.
    - @Configuration : 스프링 설정 정보를 인식하고, 스프링 빈이 싱글톤을 유지하도록 추가 처리를 한다.
    - @Service : @Service 는 특별한 처리를 하지 않는다. 대신 개발자들이 핵심 비즈니스 로직이 여기 있겠구나 라고 비즈니스 계층을 인식하는데 도움이 된다.

## 필터

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyIncludeComponent {
}

@MyIncludeComponent
public class BeanA {

}
```

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyExcludeComponent {
}

@MyExcludeComponent
public class BeanB {

}
```

```java
public class ComponentFilterAppConfigTest {

    @Test
    void filterScan() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(ComponentFilterAppConfig.class);

        ac.getBean("beanA", BeanA.class);
        Assertions.assertThat(beanA).isNotNull();

        Assertions.assesrtThrows(
            NoSuchBeanDefinitionException.class,
            () -> ac.getBean("beanB", BeanB.class);
        )
    }

    @Configuration
    @ComponentScan(
        includeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = MyIncludeComponent.class),
        excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = MyEncludeComponent.class)
        )
    static class ComponentFilterAppConfig {

    }
}
```

## 다양한 의존관계 주입 방법

- 생성자 주입
    - 이름 그대로 생성자를 통해서 의존 관계를 주입 받는 방법이다.
    - 특정
        - 생성자 호출시점에 딱 1번만 호출되는 것이 보장된다.
        - `불변, 필수` 의존 관계에 사용
    - 생성자가 한 개만 있다면 @Autowired 를 생략해도 된다.

- 수정자 주입
    - setter 라고 불리는 필드의 값을 변경하는 수정자 메서드를 통해서 의존관계를 주입하는 방법이다.
    - 특징
        - `선택, 변경` 가능성이 있는 의존관계에 사용
        - 자바빈 프로퍼티 규약의 수정자 메서드 방식을 사용하는 방법이다.

> 생성자 주입은 스프링 라이프 사이클에서 빈을 등록할 때 의존관계 주입이 발생하고, 수정자 주입은 빈을 모두 등록한 뒤, 두 번째 단계에서 의존관계 주입이 발생한다.

- 필드 주입
    - 이름 그대로 필드에 바로 주입하는 방법이다.
    - 특징
        - 코드가 간결해서 많은 개발자들을 유혹하지만 외부에서 변경이 불가능해서 테스트하기 힘들다는 치명적인 단점이 있다.
        - DI 프레임워크가 없으면 아무것도 할 수 없다.
        - `사용하지 말자!`
        - 애플리케이션의 실제 코드와 관계 없는 테스트 코트에서 사용
        - 스프링 설정을 목적으로 하는 @Configuration 같은 곳에서만 특별한 용도로 사용

## 옵션 처리

- 자동 주입 대상을 옵션으로 처리하는 방법은 다음과 같다.
    - @Autowired(required = false) : 자동 주입할 대상이 없으면 수정자 메서드 자체가 호출 안됨.
    - @Nullable : 자동 주입할 대상이 없으면 null 이 입력된다.
    - Optional<> : 자동 주입할 대상이 없으면 Optional.empty 가 입력된다.

 ## 생성자 주입을 선택해라!

 - 불변
    - 대부분의 의존관계 주입은 한 번 일어나면 애플리케이션 종료시점까지 의존 관계를 변경할 일이 없다. 오히려 대부분의 의존관계는 애플리케이션 종료 전까지 변하면 안된다.
    - 수정자 주입을 사용하면, set 메서드를 public 으로 열어두어야 한다.
    - 누군가 실수로 변경할 수도 있고, 변경하면 안되는 메서드를 열어두는 것은 좋은 설계 방법이 아니다.

- 누락
    - 생성자 주입을 사용하면 주입 데이터를 누락 했을 때 `컴파일 오류` 가 발생한다.
    - `final` 키워드 : 생성자 주입을 사용하면 필드에 final 키워드를 사용할 수 있다. 그래서 생성자에서 혹시라도 값이 설정되지 않는 오류를 컴파일 시점에 막아준다.
    
- 기억하자! 컴파일 오류는 세상에서 가장 빠르고, 좋은 오류이다!

> 참고 : 수정자 주입을 포함한 나머지 주입 방식은 모두 생성자 이후에 호출되므로, 필드에 final 키워드를 사용할 수 없다.


### REFERENCE

[김영한 님 스프링 핵심 원리 - 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8/dashboard)


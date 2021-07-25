---
layout: post
title:  "spring SQL mapper vs ORM"
summary: "spring SQL mapper vs ORM"
author: 2rohyun
date: '2021-04-15 12:42:23 +0900'
category: spring
thumbnail: /assets/img/posts/springboot.png
keywords: spring, spring boot, ORM, JPA, SQL, mapper
permalink: /blog/spring-sql-mapper-orm/
usemathjax: false
---
# spring SQL Mapper vs ORM

## SQL Mapper

직접 SQL 문을 작성하여 Data Base 에 접근하는 것
Ex ) Mybatis

## ORM(Object Relational Mapping)

DB의 데이터를 객체로 매핑시켜 데이터를 접근하는 것
Ex ) JPA, Hibernate

### Mybatis

Java에서는 DB에 접근할 수 있도록 JDBC라는 라이브러리를 제공하는데, JDBC를 사용하기 쉽게 만들어주는 것이 **Mybatis**이다. JDBC로 처리하는 부분의 일부를 코드와 파라미터 설정으로 매핑을 대신해준다.
```
장점
1. 학습이 쉽다.
2. 소스코드와 SQL을 분리할 수 있다.
단점
1. 반복적인 작업이 많아진다.
```

### JPA(Java Persistent API)

Java ORM 기술에 대한 API 표준 명세로, 이것 또한 Java에서 제공하는 API이다.
사용할 때 JPA, Spring Data JPA, Hibernate를 혼동하기 쉽다.
* JPA는 Java application에서 RDBMS를 사용하는 방식을 정의한 **인터페이스**이다. 따라서 라이브러리가 아니므로 구현 자체가 없다.
* Spring Data JPA는 JPA를 쓰기 좋게 만들어놓은 모듈이다. JPA interface를 구현해 Repository라는 인터페이스를 제공한다.
* Hibernate는 JPA의 구현체이다.
```
장점
1. CRUD 쿼리 자동 생성
2. 쿼리를 건들 필요가 없다.
단점
1. 상대적으로 학습이 어렵다.
```







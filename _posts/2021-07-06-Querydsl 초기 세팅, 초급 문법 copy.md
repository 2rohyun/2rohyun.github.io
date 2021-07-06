---
layout: post
title:  "Querydsl 초기 세팅, 초급 문법"
summary: "Querydsl"
author: 2rohyun
date: '2021-07-01 11:45:00 +0900'
category: architecture
thumbnail: /assets/img/posts/querydsl.png
keywords: spring, jpa, querydsl
permalink: /blog/querydsl-first/
usemathjax: false
---

# QueryDSL #1.

## 0. 소개

- 쿼리를 자바 코드로 작성한다.

- 동적 쿼리 문제를 깔끔하게 해결해준다.

- 쉬운 SQL 스타일 문법

- JPQL 은 쿼리가 실제로 동작하는지 런타임 시점에서 밖에 확인이 되질 않는다. QueryDSL 은 자바 코드로 작성하기 때문에 컴파일 시점에서 문법 오류를 체크할 수 있다.

- QueryDSL 은 자바 코드이기 때문에, 쿼리를 메소드로 추출하여 재사용이 가능하다.

- 따라서, 엔터프라이즈 애플리케이션을 위한 마지막 퍼즐이다! ( Spring boot + Spring Data JPA + QueryDSL )


## 1. 환경 세팅

- build.gradle 

```groovy
plugins {
    id 'org.springframework.boot' version '2.5.2'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'

    //querydsl 추가
    id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
}

group = 'study'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'

    //querydsl 추가
    implementation 'com.querydsl:querydsl-jpa'

    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.h2database:h2'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

test {
    useJUnitPlatform()
}

//querydsl 추가 시작
def querydslDir = "$buildDir/generated/querydsl"
querydsl {
    jpa = true
    querydslSourcesDir = querydslDir
}
sourceSets {
    main.java.srcDir querydslDir
}
configurations {
    querydsl.extendsFrom compileClasspath
}
compileQuerydsl {
    options.annotationProcessorPath = configurations.querydsl
}
//querydsl 추가 끝
```

- 테스트용 엔티티 생성

```java
@Entity
@Getter
@Setter
public class Hello {

    @Id @GeneratedValue
    private Long id;
}
```

- build.gradle 설정 후, build - generated - querydsl 디렉토리에 Q를 포함한 엔티티 네임.java 파일이 생성되어 있어야 한다. ( ex) QHello.java ) 

- 테스트 케이스로 실행 검증

```java
@SpringBootTest
@Transactional
class QuerydslApplicationTests {

    @Autowired
    EntityManager em;

    @Test
    void contextLoads() {

        Hello hello = new Hello();
        em.persist(hello);

        JPAQueryFactory query = new JPAQueryFactory(em);
        QHello qHello = QHello.hello;

        Hello result = query
                .selectFrom(qHello)
                .fetchOne();

        assertThat(result).isEqualTo(hello);
        assertThat(result.getId()).isEqualTo(hello.getId());
    }

}
```

## 2. Q-Type

- Q 클래스 인스턴스를 사용하는 두 가지 방법

```java
QMember qMember = new QMember("m"); // 별칭 직접 지정
QMember qMember = Qmember.member; // 기본 인스턴스 사용
```

- 별도로 선언하여 초기화하는 것 보다, select(), from(), where() 등에 static import 를 사용하는 것을 권장 !

## 3. 검색 조건 쿼리

- Querydsl 은 JPQL 이 제공하는 모든 검색 조건을 제공한다.

```java
    member.username.eq("member1") // username = 'member1'
    member.username.ne("member1") //username != 'member1'
    member.username.eq("member1").not() // username != 'member1'
    member.username.isNotNull() //이름이 is not null
    member.age.in(10, 20) // age in (10,20)
    member.age.notIn(10, 20) // age not in (10, 20)
    member.age.between(10,30) //between 10, 30
    member.age.goe(30) // age >= 30
    member.age.gt(30) // age > 30
    member.age.loe(30) // age <= 30
    member.age.lt(30) // age < 30
    member.username.like("member%") //like 검색 
    member.username.contains("member") // like ‘%member%’ 검색 
    member.username.startsWith("member") //like ‘member%’ 검색
```

## 4. 결과 조회

- fetch() : 리스트를 조회, 데이터가 없으면 빈 리스트 반환
- fetchOne() : 단 건 조회
    - 결과가 없으면 : null
    - 결과가 둘 이상이면 : com.querydsl.core.NonUniqueResultException
- fetchFirst() : limit(1).fetchOne()
- fetchResults() : 페이징 정보 포함, total count 쿼리 추가 실행
- fetchCount() : count 쿼리로 변경해서 count 수 조회

## 5. 정렬

- desc(), asc() : 일반 정렬
- nullsLast(), nullsFirst() : null 데이터 순서 부여

## 6. 페이징

- offset(long offset) : 몇 번째 row 부터 출력할지( 1번째 row 0 )
- limit(long limit) : 최대 조회 건수
- 전체 조회수가 필요할 경우 fetchResults() 를 사용하면 count 쿼리가 함께 나간다. ( 총 쿼리 2개 )

> 실무에서 페이징 쿼리를 작성할 때, 데이터를 조회하는 쿼리는 여러 테이블을 조인해야 하지만, count 쿼리는 조인이 필요없는 경우도 있다. 하지만 자동화된 count 쿼리는 원본 쿼리와 같이 모두 조인하기 때문에 성능이 안나올 수 있다. 따라서 count 쿼리에 조인이 필요없는 성능 최적화가 필요하다면, count 전용 쿼리를 별도로 작성해야 한다.


## 7. 집합

```java
public void group() throws Exception {
        List<Tuple> result = queryFactory
                .select(team.name, member.age.avg())
                .from(member)
                .join(member.team, team)
                .groupBy(team.name)
                .fetch();

        Tuple teamA = result.get(0);
        Tuple teamB = result.get(1);

        assertThat(teamA.get(team.name)).isEqualTo("teamA");
        assertThat(teamA.get(member.age.avg())).isEqualTo(15);
    }
```

- 그룹화된 결과를 제한하려면 having 절 사용!

## 8. 조인

- join(조인 대상, 별칭으로 사용할 Q타입)
- join(), innerJoin() : inner join
- leftJoin() : left outer join
- rightJoin() : right outer join

```java
 public void join() throws Exception {
        List<Member> result = queryFactory
                .selectFrom(member)
                .join(member.team, team)
                .where(team.name.eq("teamA"))
                .fetch();

        assertThat(result).extracting("username").containsExactly("member1","member2");
    }
```

- 세타 조인 : 연관관계가 없는 필드로 조인
    - from 절에 여러 엔티티를 선택해서 세타 조인
    - 외부 조인이 불가능하지만, 조인 on 을 사용할 경우 외부 조인이 가능하다.

```java
    /**
     * 세타 조인
     * 회원의 이름이 팀 이름과 같은 회원 조회
     */
    @Test
    public void theta_join() throws Exception {
        em.persist(new Member("teamA"));
        em.persist(new Member("teamB"));

        List<Member> result = queryFactory
                .select(member)
                .from(member, team)
                .where(member.username.eq(team.name))
                .fetch();

        assertThat(result)
                .extracting("username")
                .containsExactly("teamA","teamB");
    }
```

## 8. join on 절

#### 1. 조인 대상 필터링

```java
 /**
     * 예) 회원과 팀을 조인하면서, 팀 이름이 teamA인 팀만 조인, 회원은 모두 조회
     * JPQL : select m, t from Member m left join m.team t on t.name = 'teamA'
     */
    @Test
    public void join_on_filtering() throws Exception {
        List<Tuple> result = queryFactory
                .select(member, team)
                .from(member)
                .leftJoin(member.team, team).on(team.name.eq("teamA"))
                .fetch();

        for (Tuple tuple : result) {
            System.out.println("tuple = " + tuple);
        }

        //결과
        //t=[Member(id=3, username=member1, age=10), Team(id=1, name=teamA)]
        // t=[Member(id=4, username=member2, age=20), Team(id=1, name=teamA)]
        // t=[Member(id=5, username=member3, age=30), null]
        // t=[Member(id=6, username=member4, age=40), null]
    }

```

> on 절을 활용해 조인 대상을 필터링 할 때, inner join 을 사용하면 where 절에서 필터링하는 것과 기능이 동일하다. 따라서 on 절을 활용한 조인 대상 필터링을 사용할 때, inner join 은 where 절, outer join 이 필요한 경우만 on 절을 사용하자!

#### 2. 연관관계 없는 엔티티 외부 조인

```java
  /**
     * 연관관계 없는 엔티티 외부 조인
     * 회원의 이름이 팀 이름과 같은 대상 외부 조인
     */
    @Test
    public void join_on_no_relation() throws Exception {
        em.persist(new Member("teamA"));
        em.persist(new Member("teamB"));

        List<Tuple> result = queryFactory
                .select(member,team)
                .from(member)
                .leftJoin(team).on(member.username.eq(team.name))
                .fetch();

        for (Tuple tuple : result) {
            System.out.println("tuple = " + tuple);
        }
    }
```

- 주의! leftJoin() 부분에 일반 조인과 다르게 엔티티 하나만 들어간다.
    - 일반 조인 : leftJoin(member.team, team)
    - on 조인 : from(member).leftJoin(team).on(xxx)

## 9. 페치 조인

- 페치 조인은 SQL 에서 제공하는 기능이 아니다! SQL 조인을 활용해 연관된 엔티티를 한 번에 조회하는 기능이다. 주로 성능 최적화에 사용하는 방법이다.
- join(member.team, team).fetchJoin()

## 10. 서브 쿼리

```java

// 서브쿼리 goe 사용

    /**
     * 나이가 평균 이상인 회원
     */
    @Test
    public void sub_query_goe() throws Exception {
        QMember memberSub = new QMember("memberSub");

        List<Member> result = queryFactory
                .selectFrom(member)
                .where(member.age.goe(
                        select(memberSub.age.avg())
                                .from(memberSub)
                ))
                .fetch();

        assertThat(result).extracting("age").containsExactly(30, 40);
    }

// 서브쿼리 in 사용

    /**
     * 서브쿼리 여러 건 처리, in 사용
     */
    @Test
    public void sub_query_in() throws Exception {
        QMember memberSub = new QMember("memberSub");

        List<Member> result = queryFactory
                .selectFrom(member)
                .where(member.age.in(
                        select(memberSub.age)
                                .from(memberSub)
                                .where(member.age.gt(10))
                ))
                .fetch();

        assertThat(result).extracting("age").containsExactly(20, 30, 40);
    }

// select 절에 서브쿼리 사용

    @Test
    public void select_sub_query() throws Exception {
        QMember memberSub = new QMember("memberSub");

        List<Tuple> result = queryFactory
                .select(member.username,
                        select(memberSub.age.avg())
                                .from(memberSub))
                .from(member)
                .fetch();

        for (Tuple tuple : result) {
            System.out.println("tuple = " + tuple);
        }
    }
```

- JPA JPQL 서브쿼리의 한계점으로 from 절의 서브쿼리(인라인 뷰)는 지원하지 않는다. 하이버네이트 구현체를 사용하면 select 절의 서브쿼리는 지원한다. Querydsl도 하이버네이트 구현체를 사용하면 select 절의 서브쿼리를 지원한다.

- 해결방안
    1. 서브쿼리를 join 으로 변경한다. (높은 확률로 조인으로 바꿀 수 있고, 성능 상 이점을 가진다.)
    2. 애플리케이션에서 쿼리를 2번으로 분리해서 실행한다.
    3. nativeSQL을 사용한다.

> SQL 은 데이터를 가져오는 것에 집중하고, 필요하면 애플리케이션에서 로직을 태우고, 화면에서 변환해서 렌더링해야하는 것(날짜 포맷 맞추기 등)은 화면에서 해야하는 것이다. 그래야 쿼리도 재활용성이 생기고 책임을 분리하는 것이다. 즉, DB 는 데이터만 필터링, 그루핑해서 가져오고, 로직은 애플리케이션, 프레젠테이션에서 해결하는 것이 좋다.

- 책 추천 : SQL Antipatterns 

## 11. case 문

```java
 @Test
    public void basic_case() throws Exception {
        List<String> result = queryFactory
                .select(member.age
                        .when(10).then("열살")
                        .when(20).then("스무살")
                        .otherwise("기타"))
                .from(member)
                .fetch();

        for (String s : result) {
            System.out.println("s = " + s);
        }
    }

    @Test
    public void complex_case() throws Exception {
        List<String> result = queryFactory
                .select(new CaseBuilder()
                        .when(member.age.between(0, 20)).then("0~20살")
                        .when(member.age.between(21, 30)).then("21~30살")
                        .otherwise("기타"))
                .from(member)
                .fetch();

        for (String s : result) {
            System.out.println("s = " + s);
        }
    }
```

> 가급적이면 디비에서 이러한 문제들을 해결하지 않는 것이 좋다. DB 는 최소한의 필터링, 그루핑으로 데이터를 줄이는 일을 수행하고 데이터를 전환하고 바꾸고 보여주는 것은 DB 에서 하지 않는 것을 권장한다 !

## 12. 상수, 문자 더하기

```java
@Test
    public void constant() throws Exception {
        List<Tuple> result = queryFactory
                .select(member.username, Expressions.constant("A"))
                .from(member)
                .fetch();

        for (Tuple tuple : result) {
            System.out.println("tuple = " + tuple);
        }
    }

    @Test
    public void concat() throws Exception {

        //{username}_{age}
        List<String> result = queryFactory
                .select(member.username.concat("_").concat(member.age.stringValue()))
                .from(member)
                .where(member.username.eq("member1"))
                .fetch();

        for (String s : result) {
            System.out.println("s = " + s);
        }
    }
```

### REFERENCE

[김영한 님 실전! Querydsl](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84)


---
layout: post
title:  "Java Optional 기초"
summary: "java optional"
author: 2rohyun
date: '2021-04-22 12:01:23 +0900'
category: java
thumbnail: /assets/img/posts/java.png 
keywords: java, optional, null safety
permalink: /blog/java-optional/
usemathjax: false
---

# Java Optional
## 1. Optional 사용 이유
java.lang.NullPointerException을 안전하게 처리하기 위해

## 2. Optional 객체 생성
Optional 객체를 생성하기 위한 시작 연산자

1. Optional.of

```java
// 메서드 시그니처
public static <T> Optional<T> of(T value);
// 예제
Optional<String> opt = Optional.of("result");
```

value가 null 인 경우 NPE 예외 발생, 반드시 값이 있어야 하는 객체인 경우 사용

2. Optional.ofNullable

```java
// 메서드 시그니처
public static <T> Optional<T> ofNullable(T value);
// 예제
Optional<String> opt = Optional.ofNullable(null);
```

value가 null 인 경우 비어있는 Optional 객체 반환, 값이 Null 일 수도 있는 경우 사용

3. Optional.empty

```java
// 메서드 시그니처
public static<T> Optional<T> empty();
// 예제
Optional<String> emptyOpt = Optional.empty();
```

비어있는 Optional 객체 생성, 조건에 따라 분기를 태우고 반환할 값이 없는 경우 사용

`비어있는 Optional 객체란?`
```java
* 비어있는 옵셔널 객체
----------
|Optional| 이 옵셔널 객체는 생성은 되었으나 값(객체)을 가지지 않았다.
|        | 이 상태는 Optional.empty() 라고 볼 수 있다.
|        |
----------

* 값이 있는 옵셔널 객체
----------
|Optional| 이 옵셔널 객체는 생성 되고 값(객체)을 가지고 있다.
|--------|
||String||
|--------|
----------
```

## 3. Optional 중간 처리
Optional 객체를 생성한 후 사용 가능한 메소드, 다시 Optional 객체를 반환한다.

1. filter

```java
// 메서드 시그니처
public Optional<T> filter(Predicate<? super T> predicate);
// 메서드 예제 java 7 vs java 8
final List<String> names = Arrays.asList("Sehoon", "Songwoo", "Chan", "Youngsuk", "Dajung");
 
        //java 7
        System.out.println("java 7");
        final List<string> startsWithN1 = new ArrayList<string>();
        for (String name : names) {
            if (name.startsWith("S")) {
                startsWithN1.add(name);
            }
        }
 
        System.out.println(startsWithN1);
 
        System.out.println("");
 
        //java 8 Lambda
        System.out.println("java 8");
        final List<string> startsWithN2 = 
                names.stream().filter(name -> name.startsWith("S"))
                                .collect(Collectors.toList());
```

predicate 값이 참이면 해당 필터 통과, 거짓이면 통과 X

2. map

```java
// 메서드 시그니처
public<U> Optional<U> map(Function<? super T, ? extends U> mapper);
// 예제 java 7 vs java 8
final List<String> names = Arrays.asList("Sehoon", "Songwoo", "Chan", "Youngsuk", "Dajung");
            //java 7
            System.out.println("java 7");
            for(String name : names) {
                System.out.println(name.toUpperCase());
            }
 
            System.out.println("");
 
            //java 8 Lambda
            System.out.println("java 8");
            names.stream()
                .map(name -> name.toUpperCase())
                .forEach(name -> System.out.println(name));
```

mapper 함수를 통해 입력값을 다른 값으로 변환하는 메소드이다.

3. flatMap

```java
// 메서드 시그니처
public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper);
// 예제
String result = Optional.of("result")
        .flatMap((val) -> Optional.of("good"))
        .get();
System.out.println(result); // print 'good'
```

map과 flatMap 모두 쉽게 사용이 가능하며 차이점은 하나의 입력을 다시 하나의 리턴인 경우에는 map을 하나의 입력을 다시 stream 혹은 Optional로 변환을 하고자할때는 flatmap을 사용한다. 

`나중에 따로 포스트로 정리해야할 부분( map vs flatMap )`

4. stream

```java
List<String> names = Arrays.asList("jeong", "pro", "jdk", "java");
// 기존의 코딩 방식
long count = 0;
for (String name : names) {
    if (name.contains("o")) {
        count++;
    }
}
System.out.println("Count : " + count); // 2
 
// 스트림 이용한 방식
count = 0;
count = names.stream().filter(x -> x.contains("o")).count();
System.out.println("Count : " + count); // 2
```

stream은 컬렉션, 배열등의 저장 요소를 하나씩 참조하며 함수형 인터페이스(람다식)를 적용하며 반복적으로 처리할 수 있도록 해주는 기능이다.

## Optional 종단 처리
Optional 객체의 chaining 을 끝내는 메소드

1. ifPresent

```java
// 메서드 시그니처
public void ifPresent(Consumer<? super T> consumer);
// 예제1
Optional.of("test").ifPresent((value) -> {
	// something to do
});
// 예제2 (ifPresent 미수행)
Optional.ofNullable(null).ifPresent((value) -> {
	// nothing to do
});
```

2. isPresent

```java
// 메서드 시그니처
public boolean isPresent();
// 예제
Optional.ofNullable("test").isPresent(); // true
Optional.ofNullable("test").filter((val) -> "result".eqauls(val)).isPresent(); // false
```

연산이 끝난 후 객체가 존재하는 지의 여부 판단 후 boolean 반환

3. get

```java
// 메서드 시그니처
public T get();
// 예제
Optional.of("test").get(); // 'test'
Optional.ofNullable(null).get(); // NoSuchElementException!
```

4. orElse

```java
// 메서드 시그니처
public T orElse(T other);
// 예제
String result = Optional.ofNullable(null).orElse("default");
System.out.println(result); // print 'default'
```

5. orElseGet

```java
// 메서드 시그니처
public T orElseGet(Supplier<? extends T> other);
// 예제
String result = Optional.ofNullable("input").filter("test"::equals).orElseGet(() -> "default");
System.out.println(result); // print 'default'
```

6. orElseThrow

```java
// 메서드 시그니처
public T orElseThrow();
// 예제 (자바 8)
Optional.ofNullable(something).orElseThrow(NoSuchElementException::new);
// 예제 (자바 10)
Optional.ofNullable(something).orElseThrow();
```

`orElse vs orElseGet`
orElse 메서드는 옵셔널 객체가 비어있든 비어있지 않든 반드시 실행 /
orElseGet 메서드는 옵셔널 객체가 비어있으면 실행

**무조건 orElseGet 만 사용하자 !** 


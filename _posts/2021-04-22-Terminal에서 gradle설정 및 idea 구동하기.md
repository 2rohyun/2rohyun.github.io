---
layout: post
title:  "Terminal 에서 gradle 설정 & idea 구동"
summary: "terminal gradle configuration"
author: 2rohyun
date: '2021-04-22 12:01:23 +0900'
category: infra
thumbnail: /assets/img/posts/terminal.png 
keywords: infra, mac, terminal, gradle, idea
permalink: /blog/terminal-gradle-idea/
usemathjax: false
---

# Terminal에서 gradle 설정 및 idea 구동하기
### gradle 설치
```
$ brew update && brew install gradle
```

### gradle 실행
```
$ gradle init

Welcome to Gradle 6.8.3!

Here are the highlights of this release:
 - Faster Kotlin DSL script compilation
 - Vendor selection for Java toolchains
 - Convenient execution of tasks in composite builds
 - Consistent dependency resolution

For more details see https://docs.gradle.org/6.8.3/release-notes.html

Starting a Gradle Daemon (subsequent builds will be faster)

Select type of project to generate:
  1: basic
  2: application
  3: library
  4: Gradle plugin
$ Enter selection (default: basic) [1..4] 1

Select build script DSL:
  1: Groovy
  2: Kotlin
$ Enter selection (default: Groovy) [1..2] 1

Project name (default: sp-fastcampus-spring-sec):

> Task :init
Get more help with your project: Learn more about Gradle by exploring our samples at https://docs.gradle.org/6.8.3/samples

BUILD SUCCESSFUL in 5m 18s
2 actionable tasks: 2 executed
```

### IntelliJ 터미널에서 실행하기
Open IntelliJ IDEA, go to Tools->Create Command-Line Launcher… and optionally adjust the location and name of the script that will start IntelliJ IDEA. 

```
$ idea .
```

IntelliJ 실행 완료


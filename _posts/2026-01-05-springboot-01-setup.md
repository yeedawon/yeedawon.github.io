---
layout: post
title: "[Spring Boot] 01. 스프링 부트란? 환경 세팅 & 프로젝트 시작"
date: 2026-01-05
categories: [springboot]
tags: [springboot, java, initializr, gradle]
author: yeedawon
excerpt: "Spring Boot가 기존 Spring과 무엇이 다른지 알아보고, Spring Initializr로 myblog 프로젝트를 세팅합니다."
---

<div class="callout callout-src">
  <div class="callout-title">📺 참조</div>
  방구석코더 — Spring Boot 기초 강의 (2026.01.04~) &nbsp;·&nbsp;
  <a href="https://www.youtube.com/@bang_gusuk_coder/videos" target="_blank">유튜브 바로가기</a>
</div>

## Spring이란, Spring Boot란

**Spring Framework**는 자바 엔터프라이즈 애플리케이션의 사실상 표준 프레임워크입니다.  
강력하지만 `web.xml`, `applicationContext.xml` 등 대규모 XML 설정이 필수였습니다.

**Spring Boot**는 이 복잡함을 "설정보다 관례(Convention over Configuration)" 원칙으로 해결합니다.

| 항목 | Spring | Spring Boot |
|------|--------|-------------|
| 설정 방식 | XML / Java Config 직접 작성 | 자동 설정 (`@SpringBootApplication`) |
| 서버 | 외부 Tomcat 별도 설치 | 내장 Tomcat 포함 |
| 의존성 | 각 라이브러리 버전 수동 관리 | Starter 의존성으로 자동 관리 |
| 실행 | WAR → WAS 배포 | JAR 실행 (`java -jar`) |

---

## Spring Initializr로 프로젝트 생성

[start.spring.io](https://start.spring.io) 에서 아래 옵션으로 `myblog` 프로젝트를 생성했습니다.

- **Project**: Gradle - Groovy
- **Language**: Java 17
- **Spring Boot**: 3.x
- **Packaging**: Jar
- **Dependencies**: Spring Web, Spring Boot DevTools, Lombok

---

## build.gradle 살펴보기

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.0'
    id 'io.spring.dependency-management' version '1.1.4'
}

group = 'com.yeedawon'
version = '0.0.1-SNAPSHOT'

java {
    sourceCompatibility = '17'
}

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    compileOnly 'org.projectlombok:lombok'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

`spring-boot-starter-web` 하나로 Spring MVC, Tomcat, Jackson, 검증 라이브러리가 한 번에 내려받아집니다. 버전 충돌 걱정 없이 Spring이 호환 버전을 맞춰줍니다.

---

## 진입점 — `@SpringBootApplication`

```java
package com.yeedawon.myblog;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication   // ① @SpringBootConfiguration
                         // ② @EnableAutoConfiguration
                         // ③ @ComponentScan 의 합성
public class MyblogApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyblogApplication.class, args);
    }
}
```

`main()` 하나로 내장 Tomcat이 실행되고 애플리케이션이 올라갑니다.

---

## Hello World 컨트롤러

```java
package com.yeedawon.myblog.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/")
    public String hello() {
        return "Hello, DAWON's Blog!";
    }
}
```

`http://localhost:8080/` 접속 시 문자열이 바로 응답됩니다.

---

## application.properties

```properties
spring.application.name=myblog
server.port=8080
```

<div class="callout callout-git">
  <div class="callout-title">🔗 Git Reference</div>
  실습 코드: <a href="https://github.com/yeedawon/myblog" target="_blank">github.com/yeedawon/myblog</a><br>
  커밋 단계: 프로젝트 초기 세팅 (Initial commit → Hello Controller 추가)
</div>

## 정리

- Spring Boot = Spring + 자동 설정 + 내장 서버 + Starter 의존성
- `@SpringBootApplication`이 모든 것의 시작점
- Initializr로 수 분 내 프로젝트 골격 완성
- 다음 포스트: **Spring MVC 구조와 계층 분리**

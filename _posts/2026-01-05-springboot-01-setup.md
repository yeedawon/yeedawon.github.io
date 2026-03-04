---
layout: post
title: "[Spring Boot] 01. 프로젝트 세팅 — Spring Initializr & 구조 파악"
date: 2026-01-05
categories: [springboot]
tags: [springboot, java, initializr, gradle, thymeleaf]
author: yeedawon
excerpt: "Spring Initializr로 myblog 프로젝트를 생성하고, build.gradle과 YeedawonApplication을 분석합니다. Spring Boot 4.0.3 + Java 21 기반."
---

<div class="callout callout-src">
  <div class="callout-title">📺 참조 강의</div>
  방구석코더 — Spring Boot 기초 강의 (2026.01.04~) &nbsp;·&nbsp;
  <a href="https://www.youtube.com/@bang_gusuk_coder/videos" target="_blank">유튜브 채널 바로가기</a>
</div>

<div class="callout callout-git">
  <div class="callout-title">🔗 실습 코드 레포지토리</div>
  <a href="https://github.com/yeedawon/myblog" target="_blank">github.com/yeedawon/myblog</a>
  &nbsp;—&nbsp; master 브랜치 / Java 100% / 17 commits
</div>

## 왜 Spring Boot인가?

기존 Spring Framework는 강력하지만 설정이 복잡했습니다.  
**Spring Boot**는 "설정보다 관례(Convention over Configuration)" 원칙으로 이 복잡함을 해결합니다.

| 항목 | Spring Framework | Spring Boot |
|------|-----------------|-------------|
| 서버 설정 | 외부 Tomcat 별도 설치·배포 | 내장 서버 포함 |
| 의존성 관리 | 각 라이브러리 버전 수동 지정 | Starter로 자동 관리 |
| 설정 파일 | XML 대규모 설정 | application.properties 최소 설정 |
| 실행 방법 | WAR → WAS 배포 | `java -jar` 단일 명령 |
| 자동 설정 | 없음 | `@SpringBootApplication` 하나로 처리 |

---

## Spring Initializr로 프로젝트 생성

[start.spring.io](https://start.spring.io) 에서 아래 옵션으로 `yeedawon` 프로젝트를 생성했습니다.

| 항목 | 선택값 |
|------|--------|
| Project | Gradle - Groovy |
| Language | Java |
| Spring Boot | 4.0.3 |
| Group | com.blog |
| Artifact | yeedawon |
| Description | for private presentation |
| Packaging | Jar |
| Java | 21 |

**추가한 Dependencies**

| Dependency | 역할 |
|------------|------|
| Spring Web MVC | REST API, MVC 패턴 웹 개발 |
| Thymeleaf | 서버 사이드 HTML 템플릿 엔진 |
| Lombok | 보일러플레이트 코드 자동 생성 (`@Getter` 등) |
| Spring Boot DevTools | 소스 변경 시 자동 재시작 (개발 편의) |

---

## build.gradle 분석

실제 생성된 `build.gradle` 파일입니다.

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '4.0.3'
    id 'io.spring.dependency-management' version '1.1.7'
}

group = 'com.blog'
version = '0.0.1-SNAPSHOT'
description = 'for private presentation'

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
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
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    implementation 'org.springframework.boot:spring-boot-starter-webmvc'
    compileOnly 'org.projectlombok:lombok'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-thymeleaf-test'
    testImplementation 'org.springframework.boot:spring-boot-starter-webmvc-test'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

### 의존성 scope 구분

| scope | 설명 | 예시 |
|-------|------|------|
| `implementation` | 컴파일 + 런타임 모두 포함 | `spring-boot-starter-webmvc` |
| `compileOnly` | 컴파일 시에만 사용, 런타임 제외 | `lombok` (어노테이션 처리 후 불필요) |
| `developmentOnly` | 개발 환경에서만 사용, 운영 제외 | `spring-boot-devtools` |
| `annotationProcessor` | 컴파일 시 어노테이션 처리기 실행 | `lombok` |
| `testImplementation` | 테스트 코드에서만 사용 | 테스트 스타터들 |

### `io.spring.dependency-management` 플러그인

버전을 직접 명시하지 않아도 Spring Boot 버전에 맞는 호환 라이브러리 버전을  
자동으로 관리해줍니다. 예를 들어 `spring-boot-starter-webmvc` 옆에 버전 번호가  
없는 것이 바로 이 플러그인 덕분입니다.

### Java Toolchain

```groovy
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}
```

`JavaLanguageVersion.of(21)` — Java **21** LTS를 사용합니다.  
Toolchain 설정은 팀원이나 CI 환경마다 다른 JDK가 설치되어 있어도  
Gradle이 자동으로 올바른 버전의 JDK를 선택해줍니다.

---

## 진입점 — YeedawonApplication.java

```java
package com.blog.yeedawon;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class YeedawonApplication {

    public static void main(String[] args) {
        SpringApplication.run(YeedawonApplication.class, args);
    }

}
```

`@SpringBootApplication`은 아래 세 어노테이션의 합성입니다.

| 포함된 어노테이션 | 역할 |
|----------------|------|
| `@SpringBootConfiguration` | `@Configuration` 확장. 이 클래스가 Bean 설정 클래스임을 표시 |
| `@EnableAutoConfiguration` | classpath의 라이브러리를 보고 설정을 자동으로 구성 |
| `@ComponentScan` | `com.blog.yeedawon` 하위 패키지를 스캔해 Bean 등록 |

`SpringApplication.run(YeedawonApplication.class, args)` 한 줄로:
1. **ApplicationContext** (스프링 컨테이너) 생성
2. **내장 Tomcat** 서버 실행
3. 자동 설정 적용

---

## application.properties

```properties
spring.application.name=yeedawon
```

현재는 애플리케이션 이름만 지정된 최소 설정입니다.  
서버 포트를 기본값(8080)에서 바꾸거나, DB 연결 설정이 추가될 때 이 파일에 작성합니다.

---

## 프로젝트 패키지 구조

```
src/main/java/com/blog/yeedawon/
│
└── YeedawonApplication.java   ← @SpringBootApplication 진입점
```

현재는 진입점 클래스만 있는 초기 상태입니다.  
앞으로 Controller, Service, Repository 클래스들이 이 패키지 하위에 추가됩니다.

---

## 생성된 파일 구조 전체

```
yeedawon/
├── build.gradle               ← 빌드 설정 + 의존성
├── settings.gradle            ← 프로젝트 이름 설정
├── gradlew / gradlew.bat      ← Gradle Wrapper (버전 고정 실행 스크립트)
├── gradle/wrapper/
│   └── gradle-wrapper.properties
├── HELP.md                    ← Initializr 생성 참고 링크
└── src/
    ├── main/
    │   ├── java/com/blog/yeedawon/
    │   │   └── YeedawonApplication.java
    │   └── resources/
    │       ├── application.properties
    │       ├── static/        ← CSS, JS, 이미지 정적 자원
    │       └── templates/     ← Thymeleaf HTML 템플릿
    └── test/
        └── java/com/blog/yeedawon/
            └── YeedawonApplicationTests.java
```

<div class="callout callout-note">
  <div class="callout-title">💡 Gradle Wrapper란?</div>
  <code>gradlew</code> 스크립트를 통해 Gradle을 로컬에 직접 설치하지 않아도 됩니다.<br>
  <code>gradle-wrapper.properties</code>에 명시된 버전의 Gradle을 자동으로 다운로드해서 사용하므로,<br>
  팀원 모두가 동일한 빌드 환경을 보장받습니다.
</div>

## 정리

| 항목 | 값 |
|------|-----|
| Spring Boot 버전 | **4.0.3** |
| Java 버전 | **21** (LTS) |
| 빌드 도구 | Gradle (Groovy DSL) |
| 패키지 | `com.blog.yeedawon` |
| 주요 스택 | Spring Web MVC + Thymeleaf + Lombok |

- `@SpringBootApplication` = 자동 설정 + Bean 스캔 + Configuration 세 가지 합성
- `build.gradle`이 모든 의존성과 Java 버전을 관리
- `spring.application.name=yeedawon` — 최소 설정 상태
- **다음 포스트**: Controller 추가 → 첫 HTTP 요청 처리

---
layout: post
title: "[Spring Boot] 라이브러리 vs 프레임워크"
date: 2026-01-23
categories: [springboot]
tags: [library, framework, ioc, spring, java, 개념]
author: yeedawon
excerpt: "개발을 시작하면 마주치는 단어, 라이브러리와 프레임워크. 둘 다 '남이 만들어 놓은 코드'인데 왜 구분할까요? 제어의 방향이라는 핵심 개념을 중심으로 쉽게 정리합니다."
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


## 왜 이 개념이 중요한가?

개발 공부를 시작하면 금방 이런 말을 듣게 됩니다.

> "Spring은 프레임워크야." "Lombok은 라이브러리야."

둘 다 **남이 만들어 놓은 코드를 가져다 쓰는 것**인데, 왜 굳이 구분할까요?  
단순히 용어 차이가 아니라, **내 코드와 어떤 관계로 동작하는가**가 완전히 다르기 때문입니다.

---

## 가장 쉬운 비유 — 요리

<div class="diagram">  [ 라이브러리 ]                       [ 프레임워크 ]

  요리 도구                         식당(레시피 / 구축된 주방 시스템)
  레시피(라이브러리)를 필요할 때         주방 규칙(프레임워크)이 이미 정해져 있음.
  꺼내서 참고함.                         내 역할(코드)은 규칙 안에서만 수행.
</div>

---

## 라이브러리 (Library)

### 정의

> 특정 기능을 수행하는 코드의 모음. **내 코드가 필요할 때 호출해서 사용한다.**

### 특징

| 항목 | 내용 |
|------|------|
| 호출 주체 | **내 코드**가 라이브러리를 호출 |
| 흐름 제어 | **내 코드**가 전체 흐름을 결정 |
| 사용 방식 | 필요한 함수·클래스를 import 후 직접 호출 |
| 교체 가능성 | 비교적 자유롭게 교체 가능 |

### Java 예시 — Lombok

```java
import lombok.Getter;         // ← 라이브러리 import
import lombok.NoArgsConstructor;

@Getter                       // ← 내 코드가 라이브러리 기능을 '선택적으로' 사용
@NoArgsConstructor
public class Article {
    private Long id;
    private String title;
}
```

`@Getter`가 필요 없으면 그냥 안 쓰면 됩니다.  
Lombok이 내 코드 실행 흐름을 건드리지 않습니다.

### 다른 대표적인 라이브러리들

| 라이브러리 | 언어 | 역할 |
|-----------|------|------|
| Lombok | Java | 반복 코드 자동 생성 |
| Jackson | Java | JSON ↔ Java 객체 변환 |
| NumPy | Python | 수치 연산 |
| pandas | Python | 데이터 처리 |
| React | JavaScript | UI 컴포넌트 구성 |

---

## 프레임워크 (Framework)

### 정의

> 애플리케이션의 **뼈대(골격)**. 내 코드가 프레임워크가 정한 규칙과 구조 안에서 동작한다.

### 특징

| 항목 | 내용 |
|------|------|
| 호출 주체 | **프레임워크**가 내 코드를 호출 |
| 흐름 제어 | **프레임워크**가 전체 흐름을 결정 |
| 사용 방식 | 정해진 규칙(어노테이션, 클래스 상속 등)에 맞게 코드 작성 |
| 교체 가능성 | 어려움. 코드 전체가 프레임워크에 결합됨 |

### Java 예시 — Spring Boot

```java
@RestController                    // ← Spring이 이 클래스를 HTTP 처리용으로 인식
@RequestMapping("/api/articles")
public class ArticleController {

    @GetMapping("/{id}")           // ← Spring이 GET 요청이 오면 이 메서드를 '알아서 호출'
    public String getArticle(@PathVariable Long id) {
        return "article: " + id;
    }
}
```

`getArticle()` 메서드를 **내가 직접 호출하지 않았습니다.**  
HTTP GET 요청이 들어오면 **Spring이 알아서 찾아서 실행**합니다.

---

```java
// 내가 main()을 실행하면...
SpringApplication.run(YeedawonApplication.class, args);

// 이후 HTTP 요청이 오면 Spring이 알아서:
// 1. 어떤 Controller에 보낼지 판단
// 2. 해당 메서드를 직접 호출
// 3. 반환값을 HTTP 응답으로 변환
// → 이 모든 흐름을 내가 짜지 않았다 = 프레임워크
```

반면 Lombok은:

```java
// @Getter를 붙였더니 getTitle() 메서드가 생긴다
// 하지만 Lombok이 내 코드 흐름을 바꾸지는 않는다 = 라이브러리
```

---

## 정리

<div class="callout callout-note">
  <div class="callout-title">💡 한 문장 요약</div>
  <strong>라이브러리</strong>는 내가 불러서 쓰는 도구이고,
  <strong>프레임워크</strong>는 나를 불러서 쓰는 구조입니다.
</div>

| 개념 | 핵심 문장 |
|------|---------|
| 라이브러리 | 내 코드가 라이브러리를 **호출**한다 |
| 프레임워크 | 프레임워크가 내 코드를 **호출**한다 |
| 실전 | 프레임워크(Spring Boot) 안에서 라이브러리(Lombok 등)를 **함께 사용**한다 |

Spring Boot를 공부하면서 "왜 내가 메서드를 호출하지 않았는데 실행되지?"라는  
의문이 생기면, 그것이 바로 프레임워크가 작동하기 때문입니다.

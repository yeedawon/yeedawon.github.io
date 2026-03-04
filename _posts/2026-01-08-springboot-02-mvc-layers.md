---
layout: post
title: "[Spring Boot] 02. MVC 계층 분리 — Controller, Service, Repository"
date: 2026-01-08
categories: [springboot]
tags: [springboot, mvc, layered-architecture, dto]
author: yeedawon
excerpt: "Spring Boot 애플리케이션을 Controller / Service / Repository 3계층으로 분리하고, DTO 패턴을 적용합니다."
---

<div class="callout callout-src">
  <div class="callout-title">📺 참조</div>
  방구석코더 — Spring Boot 기초 강의 (2026.01.04~) &nbsp;·&nbsp;
  <a href="https://www.youtube.com/@bang_gusuk_coder/videos" target="_blank">유튜브 바로가기</a>
</div>

## 왜 계층을 나누는가?

하나의 클래스에 HTTP 처리 + 비즈니스 로직 + 데이터 저장을 모두 넣으면 유지보수가 불가능해집니다.  
Spring MVC는 **명확한 역할 분리**를 권장합니다.

```
요청
 │
 ▼
[Controller]   ← HTTP 요청/응답 처리, 유효성 검증 위임
 │
 ▼
[Service]      ← 비즈니스 로직, 트랜잭션 관리
 │
 ▼
[Repository]   ← 데이터 접근 (DB CRUD)
```

---

## 패키지 구조 설계

```
src/main/java/com/yeedawon/myblog/
├── controller/
│   └── ArticleController.java
├── service/
│   └── ArticleService.java
├── repository/
│   └── ArticleRepository.java    ← 이번엔 메모리 구현
├── dto/
│   ├── ArticleRequest.java
│   └── ArticleResponse.java
└── domain/
    └── Article.java
```

---

## Domain (Entity)

```java
package com.yeedawon.myblog.domain;

import lombok.AllArgsConstructor;
import lombok.Getter;

@Getter
@AllArgsConstructor
public class Article {
    private Long   id;
    private String title;
    private String content;
}
```

---

## DTO — Request / Response 분리

**왜 DTO를 써야 하는가?**  
Entity를 직접 외부에 노출하면 내부 구조가 API에 그대로 드러나고,  
Entity 변경이 즉시 API 변경으로 이어집니다.

```java
// ArticleRequest.java — 클라이언트 → 서버
@Getter
@NoArgsConstructor
public class ArticleRequest {
    private String title;
    private String content;
}

// ArticleResponse.java — 서버 → 클라이언트
@Getter
@AllArgsConstructor
public class ArticleResponse {
    private Long   id;
    private String title;
    private String content;

    // Entity → DTO 변환 (정적 팩토리)
    public static ArticleResponse from(Article article) {
        return new ArticleResponse(
            article.getId(),
            article.getTitle(),
            article.getContent()
        );
    }
}
```

---

## Repository — 메모리 저장소

```java
package com.yeedawon.myblog.repository;

import com.yeedawon.myblog.domain.Article;
import org.springframework.stereotype.Repository;

import java.util.*;
import java.util.concurrent.atomic.AtomicLong;

@Repository
public class ArticleRepository {

    private final Map<Long, Article> store = new HashMap<>();
    private final AtomicLong         seq   = new AtomicLong();

    public Article save(Article article) {
        Article saved = new Article(seq.incrementAndGet(),
                                    article.getTitle(),
                                    article.getContent());
        store.put(saved.getId(), saved);
        return saved;
    }

    public Optional<Article> findById(Long id) {
        return Optional.ofNullable(store.get(id));
    }

    public List<Article> findAll() {
        return new ArrayList<>(store.values());
    }

    public void deleteById(Long id) {
        store.remove(id);
    }
}
```

---

## Service

```java
package com.yeedawon.myblog.service;

import com.yeedawon.myblog.domain.Article;
import com.yeedawon.myblog.dto.ArticleRequest;
import com.yeedawon.myblog.dto.ArticleResponse;
import com.yeedawon.myblog.repository.ArticleRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.stream.Collectors;

@Service
@RequiredArgsConstructor
public class ArticleService {

    private final ArticleRepository articleRepository;

    public List<ArticleResponse> findAll() {
        return articleRepository.findAll().stream()
                .map(ArticleResponse::from)
                .collect(Collectors.toList());
    }

    public ArticleResponse findById(Long id) {
        Article article = articleRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("없는 글: " + id));
        return ArticleResponse.from(article);
    }

    public ArticleResponse save(ArticleRequest req) {
        Article article = new Article(null, req.getTitle(), req.getContent());
        return ArticleResponse.from(articleRepository.save(article));
    }

    public void delete(Long id) {
        articleRepository.deleteById(id);
    }
}
```

> Service는 비즈니스 로직만 담당합니다. HTTP 세부사항(요청/응답 객체)을 몰라야 합니다.

---

## Controller

```java
package com.yeedawon.myblog.controller;

import com.yeedawon.myblog.dto.ArticleRequest;
import com.yeedawon.myblog.dto.ArticleResponse;
import com.yeedawon.myblog.service.ArticleService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/articles")
@RequiredArgsConstructor
public class ArticleController {

    private final ArticleService articleService;

    @GetMapping
    public ResponseEntity<List<ArticleResponse>> list() {
        return ResponseEntity.ok(articleService.findAll());
    }

    @GetMapping("/{id}")
    public ResponseEntity<ArticleResponse> detail(@PathVariable Long id) {
        return ResponseEntity.ok(articleService.findById(id));
    }

    @PostMapping
    public ResponseEntity<ArticleResponse> create(@RequestBody ArticleRequest req) {
        return ResponseEntity.status(HttpStatus.CREATED)
                             .body(articleService.save(req));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        articleService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

<div class="callout callout-git">
  <div class="callout-title">🔗 Git Reference</div>
  실습 코드: <a href="https://github.com/yeedawon/myblog" target="_blank">github.com/yeedawon/myblog</a><br>
  커밋 단계: Controller/Service/Repository 패키지 분리 + DTO 적용
</div>

## 정리

- **Controller** → HTTP, **Service** → 비즈니스, **Repository** → 데이터
- Entity를 직접 반환하지 않고 **DTO로 변환**하여 외부에 노출
- `@RequiredArgsConstructor` + `final` 필드 → 생성자 주입 자동화
- 다음 포스트: **IoC / DI 원리와 Spring Bean**

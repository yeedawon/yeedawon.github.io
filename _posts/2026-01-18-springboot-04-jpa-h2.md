---
layout: post
title: "[Spring Boot] 04. JPA & H2 — 데이터베이스 연동"
date: 2026-01-18
categories: [springboot]
tags: [springboot, jpa, h2, hibernate, entity, repository]
author: yeedawon
excerpt: "Spring Data JPA로 H2 인메모리 DB와 연동합니다. @Entity, JpaRepository, 변경 감지(Dirty Checking)까지 단계별로 실습합니다."
---

<div class="callout callout-src">
  <div class="callout-title">📺 참조</div>
  방구석코더 — Spring Boot 기초 강의 (2026.01.04~) &nbsp;·&nbsp;
  <a href="https://www.youtube.com/@bang_gusuk_coder/videos" target="_blank">유튜브 바로가기</a>
</div>

## JPA란?

**JPA (Java Persistence API)** = 자바 ORM 표준 인터페이스  
**Hibernate** = JPA의 대표 구현체 (Spring Boot 기본)

```
Java 코드 (Entity)
       ↓
    JPA (인터페이스)
       ↓
  Hibernate (구현체)
       ↓
  SQL 자동 생성
       ↓
    Database
```

SQL을 직접 작성하지 않고 **자바 객체(Entity)** 로 DB를 조작합니다.

---

## 의존성 추가 (`build.gradle`)

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly   'com.h2database:h2'   // 개발용 인메모리 DB
}
```

---

## application.properties

```properties
# H2 콘솔 (개발용)
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# DataSource
spring.datasource.url=jdbc:h2:mem:myblogdb
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# JPA
spring.jpa.hibernate.ddl-auto=create-drop   # 시작 시 생성, 종료 시 삭제
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

> `ddl-auto=create-drop`은 개발 전용입니다. 운영 환경에서는 `validate` 또는 `none`을 사용하세요.

---

## Entity 클래스

```java
package com.yeedawon.myblog.domain;

import jakarta.persistence.*;
import lombok.*;

@Entity                          // DB 테이블과 매핑
@Table(name = "articles")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor
@Builder
public class Article {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)   // AUTO_INCREMENT
    private Long id;

    @Column(nullable = false, length = 255)
    private String title;

    @Column(nullable = false, columnDefinition = "TEXT")
    private String content;

    // Setter 대신 도메인 메서드
    public void update(String title, String content) {
        this.title   = title;
        this.content = content;
    }
}
```

### 주요 JPA 어노테이션

| 어노테이션 | 역할 |
|-----------|------|
| `@Entity` | 클래스 ↔ DB 테이블 매핑 |
| `@Table(name=…)` | 테이블명 직접 지정 |
| `@Id` | 기본 키(PK) 지정 |
| `@GeneratedValue` | PK 자동 생성 전략 |
| `@Column` | 컬럼 속성 (nullable, length 등) |

---

## Spring Data JPA Repository

```java
package com.yeedawon.myblog.repository;

import com.yeedawon.myblog.domain.Article;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.List;

// JpaRepository<엔티티 타입, PK 타입>
public interface ArticleRepository extends JpaRepository<Article, Long> {

    // 메서드 이름만으로 쿼리 자동 생성
    List<Article> findByTitleContaining(String keyword);
}
```

`JpaRepository`가 기본 제공하는 메서드들:

| 메서드 | 설명 |
|--------|------|
| `save(entity)` | 저장 / 수정 |
| `findById(id)` | 단건 조회 → `Optional<T>` |
| `findAll()` | 전체 조회 |
| `deleteById(id)` | 삭제 |
| `count()` | 행 수 반환 |

---

## Service — @Transactional

```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)    // 조회 기본: 읽기 전용 트랜잭션
public class ArticleService {

    private final ArticleRepository articleRepository;

    public List<ArticleResponse> findAll() {
        return articleRepository.findAll().stream()
                .map(ArticleResponse::from)
                .toList();
    }

    public ArticleResponse findById(Long id) {
        return ArticleResponse.from(getArticle(id));
    }

    @Transactional   // 쓰기 작업 재정의
    public ArticleResponse save(ArticleRequest req) {
        Article article = Article.builder()
                .title(req.getTitle())
                .content(req.getContent())
                .build();
        return ArticleResponse.from(articleRepository.save(article));
    }

    @Transactional
    public ArticleResponse update(Long id, ArticleRequest req) {
        Article article = getArticle(id);
        article.update(req.getTitle(), req.getContent());
        // save() 호출 없음! → 변경 감지(Dirty Checking)로 자동 UPDATE
        return ArticleResponse.from(article);
    }

    @Transactional
    public void delete(Long id) {
        articleRepository.deleteById(id);
    }

    private Article getArticle(Long id) {
        return articleRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("없는 글 id=" + id));
    }
}
```

### 변경 감지 (Dirty Checking)

트랜잭션 안에서 엔티티를 수정하면, 트랜잭션 커밋 시점에  
Hibernate가 최초 조회 상태와 비교해서 **자동으로 UPDATE SQL을 실행**합니다.  
`save()`를 다시 호출할 필요가 없습니다.

---

## H2 콘솔 확인

서버 실행 후 `http://localhost:8080/h2-console` 접속  
JDBC URL: `jdbc:h2:mem:myblogdb`

테이블 자동 생성, 데이터 조회를 바로 확인할 수 있습니다.

<div class="callout callout-git">
  <div class="callout-title">🔗 Git Reference</div>
  실습 코드: <a href="https://github.com/yeedawon/myblog" target="_blank">github.com/yeedawon/myblog</a><br>
  커밋 단계: spring-boot-starter-data-jpa, H2 의존성 추가 → Article Entity 정의 → JpaRepository 연동
</div>

## 정리

- JPA = 자바 ORM 표준 / Hibernate = 구현체
- `@Entity` + `@Id`로 테이블 매핑
- `JpaRepository` 상속으로 CRUD 자동 제공
- `@Transactional` + 변경 감지로 수정 자동화
- H2 콘솔로 빠른 개발·확인 사이클
- 다음 포스트: **예외 처리 & @ExceptionHandler**

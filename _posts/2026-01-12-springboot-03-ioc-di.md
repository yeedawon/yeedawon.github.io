---
layout: post
title: "[Spring Boot] 03. IoC & DI — 스프링 컨테이너와 의존성 주입"
date: 2026-01-12
categories: [springboot]
tags: [springboot, ioc, di, bean, component-scan]
author: yeedawon
excerpt: "스프링의 핵심 원리인 IoC(제어의 역전)와 DI(의존성 주입)를 이해하고, @Component, @Bean, 생성자 주입 패턴을 정리합니다."
---

<div class="callout callout-src">
  <div class="callout-title">📺 참조</div>
  방구석코더 — Spring Boot 기초 강의 (2026.01.04~) &nbsp;·&nbsp;
  <a href="https://www.youtube.com/@bang_gusuk_coder/videos" target="_blank">유튜브 바로가기</a>
</div>

## IoC — 제어의 역전

기존 방식에서는 개발자가 `new`로 직접 객체를 만들었습니다.

```java
// ❌ 기존 — 강한 결합
public class ArticleService {
    private ArticleRepository repo = new ArticleRepository(); // 개발자가 생성
}
```

Spring에서는 **IoC Container(ApplicationContext)** 가 객체를 생성·관리합니다.

```java
// ✅ Spring — 느슨한 결합
@Service
public class ArticleService {
    private final ArticleRepository repo; // 컨테이너가 주입

    public ArticleService(ArticleRepository repo) {
        this.repo = repo;
    }
}
```

객체 생성의 **제어권이 개발자 → 스프링**으로 역전되었기 때문에 *Inversion of Control*입니다.

---

## 스프링 빈 등록 방법 2가지

### ① 컴포넌트 스캔 (자동)

`@SpringBootApplication`에 포함된 `@ComponentScan`이  
`@Component` 계열 어노테이션이 붙은 클래스를 자동으로 빈으로 등록합니다.

```java
@Component      // 범용
@Service        // 비즈니스 계층 (의미론적 구분)
@Repository     // 데이터 접근 계층
@Controller     // MVC 컨트롤러
@RestController // @Controller + @ResponseBody
```

### ② `@Bean` 수동 등록

```java
@Configuration
public class AppConfig {

    @Bean
    public ArticleRepository articleRepository() {
        return new ArticleRepository();
    }

    @Bean
    public ArticleService articleService() {
        return new ArticleService(articleRepository()); // 의존성 직접 전달
    }
}
```

외부 라이브러리나 조건부 빈 등록에 주로 사용합니다.

---

## DI — 의존성 주입 3가지 방식

### ① 생성자 주입 ✅ 권장

```java
@Service
public class ArticleService {

    private final ArticleRepository repo;

    // 생성자가 1개이면 @Autowired 생략 가능
    public ArticleService(ArticleRepository repo) {
        this.repo = repo;
    }
}
```

**왜 권장?**
- `final`로 불변성 보장
- 테스트 시 Mock 주입 용이
- 순환 의존성을 컴파일 타임에 감지

실제 프로젝트에서는 Lombok을 활용합니다.

```java
@Service
@RequiredArgsConstructor   // final 필드 생성자 자동 생성
public class ArticleService {
    private final ArticleRepository repo;
    // 직접 생성자 작성 불필요
}
```

### ② 필드 주입 ⚠️

```java
@Service
public class ArticleService {
    @Autowired
    private ArticleRepository repo; // 테스트하기 어려움, 순환 의존 감지 불가
}
```

### ③ Setter 주입

```java
@Service
public class ArticleService {
    private ArticleRepository repo;

    @Autowired
    public void setRepo(ArticleRepository repo) {
        this.repo = repo;
    }
}
```

선택적 의존성에 드물게 사용합니다.

---

## 빈 스코프

| 스코프 | 설명 |
|-------|------|
| `singleton` | 기본값. 컨테이너당 1개 인스턴스 |
| `prototype` | 요청마다 새 인스턴스 |
| `request` | HTTP 요청당 1개 (웹 전용) |
| `session` | HTTP 세션당 1개 (웹 전용) |

대부분의 서비스 빈은 싱글톤으로 충분합니다.

---

## `@Qualifier` — 동일 타입 빈이 여럿일 때

```java
@Component("mainRepo")
public class ArticleRepository implements Repository { ... }

@Component("cacheRepo")
public class CacheRepository implements Repository { ... }

@Service
@RequiredArgsConstructor
public class ArticleService {

    @Qualifier("mainRepo")
    private final Repository repo;
}
```

<div class="callout callout-git">
  <div class="callout-title">🔗 Git Reference</div>
  실습 코드: <a href="https://github.com/yeedawon/myblog" target="_blank">github.com/yeedawon/myblog</a><br>
  커밋 단계: @RequiredArgsConstructor 생성자 주입으로 전환
</div>

## 정리

- **IoC**: 객체 생성·관리 권한을 스프링 컨테이너에 위임
- **DI**: 필요한 의존 객체를 컨테이너가 자동 주입
- 빈 등록: 컴포넌트 스캔(자동) vs `@Bean`(수동)
- **생성자 주입 + Lombok `@RequiredArgsConstructor`** 조합이 최선
- 다음 포스트: **JPA & H2 데이터베이스 연동**

---
layout: post
title: "[Network] 응용 계층 — HTTP, DNS, URI/URL"
date: 2026-01-28
categories: [network]
tags: [network, http, dns, uri, url, rest, aws]
author: yeedawon
excerpt: "사용자와 가장 밀접한 응용 계층의 핵심 프로토콜인 HTTP와 DNS를 정리합니다. Spring Boot REST API와 AWS에서 직접 활용되는 개념들입니다."
---

<div class="callout callout-src">
  <div class="callout-title">📚 참조</div>
  ① 혼자 공부하는 네트워크 복습 노트 (응용 계층)<br>
  ② 강의 1편: <a href="https://youtu.be/c_4x5M_GwD8?si=Mh525VtBjlHnpTYI" target="_blank">youtu.be/c_4x5M_GwD8</a><br>
  ③ 강의 2편: <a href="https://youtu.be/k7G1wXTB8Fk?si=UhoHYeayG7C_xnFK" target="_blank">youtu.be/k7G1wXTB8Fk</a>
</div>

## 응용 계층이란?

사용자와 **가장 밀접하게 닿아있는 계층**입니다.  
웹 브라우저, 이메일 클라이언트, DNS 조회 등 실제로 눈에 보이는 서비스가 여기에 속합니다.

| 프로토콜 | 기본 포트 | 전송 계층 |
|---------|---------|---------|
| HTTP | 80 | TCP |
| HTTPS | 443 | TCP |
| DNS | 53 | UDP (쿼리) / TCP (조회 결과 큰 경우) |
| FTP | 20, 21 | TCP |
| SSH | 22 | TCP |

---

## HTTP (HyperText Transfer Protocol)

### 핵심 특성

**① 요청-응답 기반 프로토콜**

HTTP는 클라이언트의 요청과 서버의 응답으로 작동합니다.  
HTTP 요청 메시지 ≠ HTTP 응답 메시지 — 서로 구조가 다릅니다.

**② 미디어 독립적 프로토콜 (Media-Independent)**

자원의 형태(HTML, JSON, 이미지, 영상)에 무관하게 동일한 인터페이스로 주고받습니다.  
자원 형태는 `Content-Type` 헤더로 표현합니다.

```
Content-Type: text/html; charset=utf-8
Content-Type: application/json
Content-Type: image/jpeg
```

`Type/Subtype` 구조입니다. 추가 설명이 필요하면 `; parameter=value` 를 붙입니다.

**③ 스테이트리스 프로토콜 (Stateless)**

HTTP는 클라이언트의 이전 상태를 기억하지 않습니다.  
각 요청을 독립적으로 처리합니다.

**왜 스테이트리스인가?**
1. 모든 클라이언트 상태를 유지하면 서버 부담이 과도함
2. 특정 서버에 종속되지 않아 확장성(Scalability) 향상
3. 서버 하나가 다운돼도 다른 서버로 대체 가능 → 견고성(Robustness)

> **상태가 필요한 경우**: Cookie, Session, JWT 토큰으로 상태를 클라이언트 측에서 관리

**④ 지속 연결 (Persistent Connection / Keep-Alive)**

HTTP/1.1부터 기본 지원합니다.  
하나의 TCP 연결로 여러 요청-응답을 처리합니다.  
매 요청마다 3-way handshake를 반복하지 않아 성능이 크게 향상됩니다.

---

## HTTP 메시지 구조

```
Start Line (요청: Request Line / 응답: Status Line)
Field Line × n (헤더)  →  0개 이상
(빈 줄)
Message Body (페이로드)  →  선택적
```

### 요청 메시지 — Request Line

```
Method   Request-Target   HTTP-Version  (줄바꿈)
GET      /api/articles    HTTP/1.1
```

**HTTP 메서드**

| 메서드 | 의미 | Spring Boot |
|--------|------|------------|
| GET | 자원 조회 | `@GetMapping` |
| POST | 자원 생성 | `@PostMapping` |
| PUT | 자원 전체 수정 | `@PutMapping` |
| PATCH | 자원 일부 수정 | `@PatchMapping` |
| DELETE | 자원 삭제 | `@DeleteMapping` |

### 응답 메시지 — Status Line

```
HTTP-Version   Status-Code   Reason-Phrase  (줄바꿈)
HTTP/1.1       200           OK
```

**주요 HTTP 상태 코드**

| 코드 | 의미 | 사용 시점 |
|------|------|----------|
| 200 OK | 성공 | GET, PUT 성공 |
| 201 Created | 생성 성공 | POST 성공 |
| 204 No Content | 응답 본문 없음 | DELETE 성공 |
| 301 Moved Permanently | 영구 이동 | URL 변경 |
| 400 Bad Request | 잘못된 요청 | 유효성 검사 실패 |
| 401 Unauthorized | 인증 실패 | 로그인 필요 |
| 403 Forbidden | 권한 없음 | 접근 거부 |
| 404 Not Found | 자원 없음 | 없는 ID 조회 |
| 500 Internal Server Error | 서버 오류 | 예상치 못한 에러 |

---

## URI, URL, URN

**URI (Uniform Resource Identifier)**: 자원을 **식별**할 수 있는 정보

```
URI
 ├── URL — 자원의 위치(어떻게 접근할지)
 └── URN — 자원의 고유 이름
```

### URL 구조

```
foo://www.example.com:8042/over/there?name=ferret#nose
 │         │          │       │           │        │
scheme   authority   port   path        query   fragment
(HTTP)   (호스트)         (경로)    (쿼리 파라미터) (앵커)
```

**실제 예시**

```
https://api.example.com/articles?category=spring&sorted=true#section1

scheme   = https
authority = api.example.com
path      = /articles
query     = category=spring&sorted=true   ← key=value 형태의 Dictionary
fragment  = section1                       ← HTML 특정 위치 이동
```

**쿼리 파라미터 (Query String)**

Spring Boot에서 `@RequestParam`으로 받는 값입니다.

```java
// GET /articles?category=spring&page=1
@GetMapping("/articles")
public List<ArticleResponse> list(
    @RequestParam String category,
    @RequestParam(defaultValue = "1") int page) { ... }
```

---

## DNS (Domain Name System)

IP 주소만으로 수신지를 특정하기 어려워 만든 프로토콜입니다.  
**도메인 이름 → IP 주소로 변환(resolve)** 합니다.

```
예: www.example.com → 203.0.113.1
```

`hosts` 파일로 개인 도메인↔IP 매핑을 수동 지정할 수도 있습니다.

### DNS 계층 구조

```
www . example . com .
 │       │       │   └── 루트 도메인 (보통 생략)
 │       │       └────── TLD (최상위 도메인, .com/.net/.kr 등)
 │       └────────────── 2단계 도메인
 └────────────────────── 3단계(서브) 도메인
```

**FQDN (Fully Qualified Domain Name)**: `www.example.com.` 과 같이 루트 도메인까지 포함한 전체 도메인 이름

### DNS 질의 과정

```
클라이언트
   │── ① 로컬 네임 서버에 질의 (ISP 제공 / 또는 8.8.8.8)
         └── ② 루트 네임 서버 질의 (루트 도메인 관장)
                  └── ③ TLD 네임 서버 질의 (.com 관장)
                           └── ④ 책임(권한) 네임 서버 질의 (example.com 관장)
                                    └── IP 주소 반환
```

**재귀적 질의**: 로컬 네임 서버가 대신 모든 단계를 거쳐 결과를 클라이언트에 반환  
**반복적 질의**: 클라이언트가 각 서버에 직접 단계별로 질의

**DNS 캐시**: 네임 서버가 응답 결과를 TTL 동안 임시 저장합니다.  
같은 질의가 오면 캐시에서 바로 반환 → 루트 네임 서버 과부하 방지

---

## ICMP (Internet Control Message Protocol)

IP 패킷 전달 과정의 오류나 진단 정보를 전달합니다.  
`ping` 명령어가 ICMP를 사용합니다.

```
ping google.com   →   ICMP Echo Request / Echo Reply
traceroute        →   ICMP Time Exceeded (TTL 만료)
```

**ICMP = IP 보조 프로토콜** (신뢰성 완전 보장 X)

---

## 백엔드 / AWS에서의 응용 계층

| 개념 | AWS / Spring Boot 적용 |
|------|----------------------|
| HTTP 요청/응답 | Spring Boot REST API (`@RestController`) |
| 상태 코드 | `ResponseEntity.status(HttpStatus.CREATED)` |
| DNS 조회 | Route 53 (AWS의 DNS 서비스) |
| HTTPS | ACM(인증서) + ALB로 SSL 종료 |
| 지속 연결 | Spring Boot 내장 Tomcat의 Keep-Alive 기본 지원 |
| URI 설계 | RESTful API: `/articles/{id}` |
| 쿼리 파라미터 | `@RequestParam` |

### Spring Boot에서 HTTP 상태 코드 반환

```java
@PostMapping("/articles")
public ResponseEntity<ArticleResponse> create(@RequestBody ArticleRequest req) {
    ArticleResponse response = articleService.save(req);
    return ResponseEntity
            .status(HttpStatus.CREATED)   // 201
            .body(response);
}

@DeleteMapping("/articles/{id}")
public ResponseEntity<Void> delete(@PathVariable Long id) {
    articleService.delete(id);
    return ResponseEntity.noContent().build();  // 204
}
```

<div class="callout callout-src">
  <div class="callout-title">📝 학습 메모</div>
  HTTP Stateless 개념이 Spring Boot API 설계에 직결됩니다.<br>
  서버가 상태를 저장하지 않으므로 인증 정보(JWT 토큰 등)를 매 요청마다 Header에 실어야 합니다.<br>
  AWS Route 53으로 도메인 등록 시 DNS 질의 흐름이 정확히 이 과정을 거칩니다.
</div>

## 정리

- **HTTP**: 요청-응답 기반, 미디어 독립적, **Stateless**, 지속 연결
- **HTTP 메서드**: GET/POST/PUT/PATCH/DELETE → Spring Boot `@XxxMapping` 직접 대응
- **상태 코드**: 200/201/204/400/401/403/404/500 필수 암기
- **URI/URL**: scheme + authority + path + query + fragment
- **DNS**: 도메인 → IP 변환, 캐시로 성능 최적화, AWS Route 53
- **ICMP**: `ping`, `traceroute` 진단 도구

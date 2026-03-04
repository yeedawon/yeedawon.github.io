---
layout: post
title: "[Network] 응용 계층 — HTTP, DNS, URI"
date: 2026-02-20
categories: [network]
tags: [network, http, dns, uri, rest, aws]
author: yeedawon
excerpt: "사용자와 가장 가까운 응용 계층. HTTP 요청/응답 구조, 상태 코드, DNS 조회 흐름, URI 구조를 표와 다이어그램으로 정리합니다."
---

<div class="callout callout-src">
  <div class="callout-title">📚 참조</div>
  ① 혼자 공부하는 네트워크 복습 노트<br>
  ② 강의 1편: <a href="https://youtu.be/c_4x5M_GwD8?si=Mh525VtBjlHnpTYI" target="_blank">youtu.be/c_4x5M_GwD8</a> &nbsp;
  ③ 강의 2편: <a href="https://youtu.be/k7G1wXTB8Fk?si=UhoHYeayG7C_xnFK" target="_blank">youtu.be/k7G1wXTB8Fk</a>
</div>

## 응용 계층이란?

TCP/IP 4계층 중 사용자와 **직접 맞닿은 최상위 계층**입니다.
HTTP, DNS, FTP, SSH 등 실제로 눈에 보이는 서비스가 모두 여기에 속합니다.

**주요 응용 계층 프로토콜**

| 프로토콜 | 포트 | 전송 계층 | 역할 |
|---------|------|---------|------|
| **HTTP** | 80 | TCP | 웹 |
| **HTTPS** | 443 | TCP | 보안 웹 |
| **DNS** | 53 | UDP (조회) / TCP (큰 응답) | 도메인 → IP 변환 |
| SSH | 22 | TCP | 원격 접속 |
| FTP | 20, 21 | TCP | 파일 전송 |
| DHCP | 67, 68 | UDP | IP 자동 할당 |
| SMTP | 25 | TCP | 이메일 전송 |

---

## HTTP (HyperText Transfer Protocol)

### 네 가지 핵심 특성

**① 요청-응답 기반**

클라이언트가 요청을 보내면 서버가 응답합니다. 서버가 먼저 보낼 수 없습니다.
(단, WebSocket은 양방향 통신 가능)

**② 미디어 독립적 (Media-Independent)**

자원의 형태와 무관하게 동일한 인터페이스로 주고받습니다.
자원 형태는 `Content-Type` 헤더로 알립니다.

| Content-Type | 의미 |
|-------------|------|
| `text/html; charset=utf-8` | HTML 문서 |
| `application/json` | JSON 데이터 (REST API 응답) |
| `image/jpeg` | JPEG 이미지 |
| `multipart/form-data` | 파일 업로드 |

**③ 스테이트리스 (Stateless)**

서버는 클라이언트의 이전 요청을 기억하지 않습니다. 각 요청은 독립적입니다.

| 항목 | 설명 |
|------|------|
| 장점 | 서버 확장 용이, 특정 서버에 종속되지 않음 |
| 단점 | 상태 유지를 위한 별도 수단 필요 |
| 해결 방법 | Cookie, Session, **JWT 토큰** (매 요청 헤더에 포함) |

**④ 지속 연결 (Persistent Connection / Keep-Alive)**

HTTP/1.1부터 기본값입니다. 하나의 TCP 연결로 여러 요청·응답을 처리합니다.

<img src="/assets/images/network/post3-http-dns-uri/http-persistent.svg" alt="HTTP 지속 연결 비교" style="max-width: 100%;">

---

## HTTP 메시지 구조

### 요청 메시지 (Request)

<img src="/assets/images/network/post3-http-dns-uri/http-request.svg" alt="HTTP 요청 메시지 구조" style="max-width: 100%;">

### HTTP 메서드

| 메서드 | 의미 | 멱등성 | 주로 사용하는 상황 |
|--------|------|-------|----------------|
| **GET** | 자원 조회 | ✅ | 목록 조회, 단건 조회 |
| **POST** | 자원 생성 | ❌ | 새 데이터 등록 |
| **PUT** | 자원 전체 수정 | ✅ | 데이터 전체 교체 |
| **PATCH** | 자원 일부 수정 | ❌ | 특정 필드만 수정 |
| **DELETE** | 자원 삭제 | ✅ | 데이터 삭제 |

> **멱등성**: 같은 요청을 여러 번 해도 결과가 동일하면 멱등(idempotent)

### 응답 메시지 (Response)

<img src="/assets/images/network/post3-http-dns-uri/http-response.svg" alt="HTTP 응답 메시지 구조" style="max-width: 100%;">

### HTTP 상태 코드

| 분류 | 범위 | 의미 |
|------|------|------|
| 1xx | 100~199 | 정보 (요청 처리 중) |
| 2xx | 200~299 | 성공 |
| 3xx | 300~399 | 리다이렉션 |
| 4xx | 400~499 | 클라이언트 오류 |
| 5xx | 500~599 | 서버 오류 |

**자주 쓰이는 상태 코드 — Spring Boot REST API에서 사용**

| 코드 | 이름 | 사용 시점 |
|------|------|---------|
| **200** OK | 성공 | GET 성공, PUT 성공 |
| **201** Created | 생성 성공 | POST 성공 |
| **204** No Content | 응답 본문 없음 | DELETE 성공 |
| **301** Moved Permanently | 영구 이동 | URL 변경 |
| **400** Bad Request | 잘못된 요청 | 유효성 검사 실패 |
| **401** Unauthorized | 인증 실패 | 로그인 필요 |
| **403** Forbidden | 권한 없음 | 접근 거부 |
| **404** Not Found | 자원 없음 | 없는 ID 조회 |
| **500** Internal Server Error | 서버 오류 | 예상치 못한 예외 |

---

## URI, URL, URN

<img src="/assets/images/network/post3-http-dns-uri/uri-url-urn.svg" alt="URI, URL, URN 관계" style="max-width: 100%;">

### URL 구조 분해

| 구성 요소 | 예시 | 설명 |
|---------|------|------|
| scheme | `https` | 프로토콜 지정 |
| authority | `api.example.com` | 호스트명 (도메인 또는 IP) |
| port | `:8080` | 생략 시 scheme 기본 포트 (http=80, https=443) |
| path | `/articles` | 자원 경로 |
| query | `?category=spring&page=1` | key=value 형태, `&`로 구분 |
| fragment | `#section2` | HTML 특정 위치 이동 (서버로 전달 안 됨) |

---

## DNS (Domain Name System)

IP 주소만으로 서버를 기억하기 어려워서, **도메인 이름 → IP 주소** 로 변환합니다.

**주요 TLD 종류**

| TLD | 의미 |
|-----|------|
| .com | 상업적 목적 (일반) |
| .net | 네트워크 관련 |
| .org | 비영리 기관 |
| .kr | 한국 ccTLD |
| .io | 개발자/스타트업 선호 |

### DNS 질의 흐름 (재귀적 조회)

<img src="/assets/images/network/post3-http-dns-uri/dns-query.svg" alt="DNS 질의 흐름" style="max-width: 100%;">

**DNS 캐시와 TTL**

| 항목 | 설명 |
|------|------|
| DNS 캐시 | 조회 결과를 TTL 동안 임시 저장 |
| TTL | 캐시 유효 시간 (짧으면 빠른 반영, 길면 부하 감소) |
| 효과 | 루트 네임서버 과부하 방지, 조회 속도 향상 |


## ICMP (Internet Control Message Protocol)

IP 패킷 전달 과정의 **오류 메시지와 진단 정보**를 전달합니다.
데이터 전송용이 아닌 네트워크 상태 확인용 보조 프로토콜입니다.

| ICMP 메시지 타입 | 의미 | 활용 도구 |
|---------------|------|---------|
| Echo Request / Reply | 호스트 도달 여부 확인 | `ping` |
| Time Exceeded | TTL 만료로 패킷 폐기 | `traceroute` |
| Destination Unreachable | 목적지 도달 불가 | 오류 진단 |
| Redirect | 더 나은 경로 안내 | 라우터 경로 최적화 |

---

**API 설계와 HTTP 상태 코드 매핑 예시**

| API | 메서드 | 성공 응답 코드 | 실패 응답 코드 |
|-----|--------|------------|------------|
| 글 목록 조회 | GET | 200 OK | 500 Internal Server Error |
| 글 단건 조회 | GET | 200 OK | 404 Not Found |
| 글 등록 | POST | 201 Created | 400 Bad Request |
| 글 수정 | PUT | 200 OK | 404 Not Found |
| 글 삭제 | DELETE | 204 No Content | 404 Not Found |

## 정리

| 개념 | 핵심 |
|------|------|
| HTTP | 요청-응답, 미디어 독립, **Stateless**, Keep-Alive |
| HTTP 메서드 | GET/POST/PUT/PATCH/DELETE |
| URI / URL | scheme + authority + port + path + query + fragment |
| DNS | 도메인 → IP, 계층적 조회 후 캐시 저장 |
| ICMP | 네트워크 진단 프로토콜 |

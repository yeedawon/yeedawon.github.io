---
layout: post
title: "[Network] 전송 계층 — TCP, UDP, 3-way Handshake, 흐름·혼잡 제어"
date: 2026-02-15
categories: [network]
tags: [network, tcp, udp, handshake, flow-control, congestion-control]
author: yeedawon
excerpt: "포트 번호로 프로세스를 구분하고, TCP의 신뢰성 보장 메커니즘(오류·흐름·혼잡 제어)과 3-way Handshake를 다이어그램으로 정리합니다."
---

<div class="callout callout-src">
  <div class="callout-title">📚 참조</div>
  ① 혼자 공부하는 네트워크 복습 노트<br>
  ② 강의 1편: <a href="https://youtu.be/c_4x5M_GwD8?si=Mh525VtBjlHnpTYI" target="_blank">youtu.be/c_4x5M_GwD8</a> &nbsp;
  ③ 강의 2편: <a href="https://youtu.be/k7G1wXTB8Fk?si=UhoHYeayG7C_xnFK" target="_blank">youtu.be/k7G1wXTB8Fk</a>
</div>

## 전송 계층의 역할

네트워크 계층(IP)이 **호스트 간** 통신을 담당한다면,
전송 계층은 **호스트 내 특정 애플리케이션 프로세스**까지 데이터를 전달합니다.

<img src="/assets/images/network/post2-tcp-udp/1-host-port.svg" alt="호스트와 포트 구조" style="max-width: 100%;">

---

## 포트 (Port)

**포트**: 같은 호스트 내에서 여러 애플리케이션을 구분하는 번호 (16 bit, 0 ~ 65,535)

| 범위 | 구분 | 예시 |
|------|------|------|
| 0 ~ 1,023 | 잘 알려진 포트 (Well-Known) | HTTP:80, HTTPS:443, SSH:22 |
| 1,024 ~ 49,151 | 등록된 포트 (Registered) | MySQL:3306, Spring Boot:8080 |
| 49,152 ~ 65,535 | 동적 포트 (Ephemeral) | 클라이언트 측 임시 포트 |

**주요 포트 번호 — AWS / 백엔드 필수**

| 포트 | 프로토콜 | 설명 |
|------|---------|------|
| 22 | SSH | EC2 원격 접속 |
| 80 | HTTP | 웹 서버 |
| 443 | HTTPS | 보안 웹 서버 |
| 3306 | MySQL | RDS 데이터베이스 |
| 8080 | HTTP-alt | Spring Boot 기본 포트 |
| 53 | DNS | 도메인 이름 조회 |
| 67/68 | DHCP | IP 자동 할당 |

---

## TCP vs UDP

| 항목 | TCP | UDP |
|------|-----|-----|
| 연결 방식 | 연결형 (3-way handshake 후 통신) | 비연결형 (바로 전송) |
| 신뢰성 | 높음 (순서 보장 + 재전송) | 낮음 (유실 시 재전송 없음) |
| 속도 | 상대적으로 느림 | 빠름 |
| 헤더 크기 | 20 ~ 60 byte | 8 byte (고정) |
| 흐름/혼잡 제어 | 있음 | 없음 |
| 사용 예 | HTTP, HTTPS, FTP, SSH, DB | DNS, DHCP, 동영상 스트리밍, 온라인 게임 |

---

## TCP 세그먼트 헤더

<img src="/assets/images/network/post2-tcp-udp/2-tcp-header.svg" alt="TCP 세그먼트 헤더 구조" style="max-width: 100%;">

**주요 제어 비트(플래그)**

| 비트 | 이름 | 의미 |
|------|------|------|
| SYN | Synchronize | 연결 수립 요청 |
| ACK | Acknowledgement | 수신 확인 |
| FIN | Finish | 연결 종료 요청 |
| RST | Reset | 연결 강제 초기화 |
| PSH | Push | 버퍼링 없이 즉시 전달 |

---

## 연결 수립 — 3-way Handshake

<img src="/assets/images/network/post2-tcp-udp/3-handshake-3way.svg" alt="3-way Handshake" style="max-width: 100%;">

---

## 연결 종료 — 4-way Handshake

<img src="/assets/images/network/post2-tcp-udp/4-handshake-4way.svg" alt="4-way Handshake" style="max-width: 100%;">

> **TIME_WAIT**: A가 보낸 마지막 ACK가 유실되었을 때 B의 FIN 재전송을 받을 수 있도록 잠시 대기합니다.

---

## 오류 제어 — ARQ 방식 비교

**ARQ (Automatic Repeat reQuest)**: 오류 감지 시 자동으로 재전송하는 메커니즘

### Stop-and-Wait ARQ

<img src="/assets/images/network/post2-tcp-udp/5-arq-stop-wait.svg" alt="Stop-and-Wait ARQ" style="max-width: 100%;">

| 장점 | 단점 |
|------|------|
| 구현 단순, 신뢰성 높음 | 네트워크 자원 낭비, 처리량 낮음 |

### Go-Back-N ARQ (파이프라이닝)

<img src="/assets/images/network/post2-tcp-udp/6-arq-gobackn.svg" alt="Go-Back-N ARQ" style="max-width: 100%;">

| 장점 | 단점 |
|------|------|
| Stop-and-Wait보다 처리량 높음 | 오류 없는 세그먼트도 재전송 |

### Selective Repeat ARQ

<img src="/assets/images/network/post2-tcp-udp/7-arq-selective.svg" alt="Selective Repeat ARQ" style="max-width: 100%;">

| 장점 | 단점 |
|------|------|
| 불필요한 재전송 없음, 효율 최고 | 수신 버퍼 관리 복잡 |

---

## 흐름 제어 — 슬라이딩 윈도우

수신자가 처리할 수 있는 데이터량을 **수신 윈도우(rwnd)** 로 알려주면,
송신자는 ACK 없이 그 크기만큼 연속으로 보낼 수 있습니다.

<img src="/assets/images/network/post2-tcp-udp/8-sliding-window.svg" alt="슬라이딩 윈도우" style="max-width: 100%;">

수신자 버퍼가 가득 찰수록 `rwnd` 값이 줄어들고, 비면 다시 늘어납니다.

---

## 혼잡 제어

네트워크 자체의 혼잡(패킷 손실·지연)을 막기 위해 **송신량을 조절**합니다.

**혼잡 감지 신호**

| 신호 | 의미 | 대응 |
|------|------|------|
| ACK 3번 중복 수신 | 특정 세그먼트 손실 (경미한 혼잡) | 빠른 재전송 (Fast Retransmit) |
| 타임아웃 발생 | 심각한 혼잡 | 혼잡 윈도우 1로 리셋 |

### 슬로 스타트 + AIMD 흐름

<img src="/assets/images/network/post2-tcp-udp/9-slow-start.svg" alt="슬로 스타트 + AIMD" style="max-width: 100%;">

**혼잡 제어 상태 전환 요약**

| 현재 상태 | 이벤트 | 다음 상태 |
|---------|-------|---------|
| 슬로 스타트 | cwnd < ssthresh | cwnd × 2 (지수 증가) |
| 슬로 스타트 | cwnd ≥ ssthresh | 혼잡 회피(선형 증가)로 전환 |
| 혼잡 회피 | ACK 3중 중복 | ssthresh = cwnd/2, 빠른 복구 |
| 혼잡 회피 | 타임아웃 | cwnd = 1, ssthresh = cwnd/2, 슬로 스타트 재시작 |

---

## AWS / Spring Boot에서 TCP 적용

| 개념 | 실제 적용 |
|------|---------|
| 포트 22 | EC2 SSH 접속 시 보안 그룹 인바운드 허용 필요 |
| 포트 8080 | Spring Boot 앱 배포 시 보안 그룹 개방 |
| 포트 3306 | RDS MySQL 연결 (프라이빗 서브넷 내부만 허용) |
| 3-way Handshake | HTTP 요청마다 TCP 연결 수립 (HTTP/1.1 keep-alive로 재사용) |
| 4-way Handshake | 연결 종료, TIME_WAIT은 서버 성능에 영향 가능 |
| ALB (로드밸런서) | 클라이언트↔ALB, ALB↔EC2 각각 독립된 TCP 연결 |

## 정리

| 개념 | 핵심 |
|------|------|
| 포트 | 호스트 내 프로세스 식별 (IP + 포트 = 소켓) |
| TCP | 연결형, 신뢰성, 오류·흐름·혼잡 제어 |
| UDP | 비연결형, 빠름, 단순 |
| 3-way Handshake | SYN → SYN+ACK → ACK |
| 슬라이딩 윈도우 | 흐름 제어, 수신 버퍼 크기만큼 연속 전송 |
| 슬로 스타트 + AIMD | 혼잡 제어 기본 알고리즘 |

다음 포스트: **응용 계층 — HTTP, DNS, URI**

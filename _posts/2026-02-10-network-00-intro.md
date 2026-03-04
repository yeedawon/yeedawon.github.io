---
layout: post
title: "[Network] 시리즈 소개 — 네트워크 기초부터 응용 계층까지"
date: 2026-02-10
categories: [network]
tags: [network, study, intro]
author: yeedawon
excerpt: "「혼자 공부하는 네트워크」 강의와 직접 필기한 노트를 기반으로, 생성형 AI를 활용해 정리한 네트워크 시리즈의 소개 글입니다."
---

<div class="callout callout-info">
  <div class="callout-title">📝 이 시리즈에 대하여</div>
  이 네트워크 시리즈는 <strong>「혼자 공부하는 네트워크」</strong> 교재와 관련 강의를 수강하며 작성한
  <strong>손필기 복습 노트</strong>를 원본 자료로 하여, <strong>생성형 AI(Claude)</strong>를 활용해
  구조화·요약한 포스트입니다.<br><br>
  원본 노트의 내용을 바탕으로 AI가 표·다이어그램·코드 형태로 재구성했으며,
  AWS 실무 연결 부분은 학습 과정에서 추가로 정리한 내용입니다.
</div>

## 원본 자료

이 시리즈의 바탕이 된 자료는 아래와 같습니다.

| 구분 | 내용 |
|------|------|
| 교재 | 혼자 공부하는 네트워크 (한빛미디어) |
| 강의 ① | [youtu.be/c_4x5M_GwD8](https://youtu.be/c_4x5M_GwD8?si=Mh525VtBjlHnpTYI){:target="_blank"} |
| 강의 ② | [youtu.be/k7G1wXTB8Fk](https://youtu.be/k7G1wXTB8Fk?si=UhoHYeayG7C_xnFK){:target="_blank"} |
| 원본 노트 | 수기 복습 노트 (PDF) |
| 정리 도구 | 생성형 AI (Claude) — 표·다이어그램 구조화, 내용 요약 |

## 시리즈 구성

| # | 포스트 | 핵심 주제 |
|---|--------|---------|
| 1 | [네트워크 계층 — IP 주소, 서브네팅, 라우팅, NAT, DHCP](/network/2026/01/22/network-01-ip-routing-nat/) | IPv4, CIDR, 공인/사설 IP, NAT, ARP, DHCP |
| 2 | [전송 계층 — TCP, UDP, 3-way Handshake, 흐름·혼잡 제어](/network/2026/01/25/network-02-tcp-udp/) | 포트, TCP vs UDP, Handshake, ARQ, 슬라이딩 윈도우, 혼잡 제어 |
| 3 | [응용 계층 — HTTP, DNS, URI](/network/2026/01/28/network-03-http-dns-uri/) | HTTP 메시지, 상태 코드, DNS 조회, URI/URL 구조 |

## AI 활용 방식

이 시리즈에서 생성형 AI는 다음과 같은 역할로 활용되었습니다.

| 역할 | 설명 |
|------|------|
| 구조화 | 손필기 노트의 흐름을 계층별 포스트로 분리·재배치 |
| 표 변환 | 텍스트 나열 형태의 비교 항목을 마크다운 표로 정리 |
| 다이어그램 생성 | 노트 속 손그림을 SVG 다이어그램으로 재작성 |
| AWS 연결 | 각 개념이 AWS 환경에서 어떻게 적용되는지 매핑 |

원본 노트의 학습 내용을 충실히 반영하되, AI의 구조화 능력을 활용해 가독성과 참조 편의성을 높이는 데 초점을 두었습니다.

---

> 첫 번째 포스트: **[네트워크 계층 — IP 주소, 서브네팅, 라우팅, NAT, DHCP →](/network/2026/01/22/network-01-ip-routing-nat/)**

---
title: 브라우저 네트워크 요청 흐름
aliases: [네트워크 요청 흐름, Request Flow, 요청 계층]
tags:
  - browser
  - networking
  - service-worker
  - cdn
  - middleware
  - caching
  - performance
created: 2026-05-12
updated: 2026-05-12
reviewed: false
---

## 정의

사용자 액션부터 서버 응답까지, 요청이 거치는 전체 네트워크 계층과 각 계층의 역할. 각 계층은 가능한 한 빨리 응답을 반환해서 다음 계층으로 보내지 않는 것이 성능 최적화의 핵심이다.

## 상세 설명

### 전체 흐름

```
[사용자 액션 (클릭, 네비게이션)]
  │
  ▼
[브라우저 캐시 (HTTP Cache)]
  │  Cache-Control, ETag 등으로 판단
  │  히트 → 바로 응답 반환
  │
  ▼
[Service Worker]
  │  fetch 이벤트로 가로챔
  │  Cache API 히트 → 바로 응답 반환
  │  미스 → 네트워크로 전달
  │
  ▼
[네트워크 (DNS → TCP → TLS)]
  │
  ▼
[CDN / Edge]
  │  정적 자산 캐시 히트 → 바로 응답 반환
  │
  ▼
[로드 밸런서]
  │
  ▼
[Next.js Middleware (Edge Runtime)]
  │  인증 체크, 리다이렉트, 헤더 조작
  │  리다이렉트 → 새 응답 반환
  │
  ▼
[서버 (Node.js Runtime)]
  │  SSR / API Route / RSC 처리
  │
  ▼
[응답 — 역순으로 돌아감]
  │
[Next.js Middleware] → 응답 헤더 수정 가능
  ▼
[CDN] → 응답 캐싱
  ▼
[Service Worker] → Cache API에 응답 저장 가능
  ▼
[브라우저 캐시] → HTTP 캐시 저장
  ▼
[렌더링 → 화면 표시]
```

### 각 계층에서 끊기는 조건

| 계층 | 끊기는 조건 | 예시 |
|---|---|---|
| 브라우저 캐시 | `Cache-Control: max-age` 유효 | 이미 받은 CSS/JS |
| Service Worker | Cache API에 있음 | 오프라인 대응, 프리캐시 자산 |
| CDN | Edge에 캐시됨 | 이미지, 정적 페이지 |
| Middleware | 리다이렉트/차단 | 미인증 사용자 → `/login` |

### Service Worker vs Next.js Middleware

둘 다 "요청을 가로챈다"는 점은 같지만, 위치가 네트워크의 양쪽 끝으로 정반대다.

| | Service Worker | Next.js Middleware |
|---|---|---|
| 실행 위치 | 브라우저 (클라이언트) | 서버 (Edge Runtime) |
| 가로채는 대상 | 브라우저에서 나가는 요청 | 서버에 들어오는 요청 |
| 시점 | 요청이 서버로 가기 전 | 서버가 응답을 만들기 전 |
| 주요 역할 | 캐싱, 오프라인 | 인증, 리다이렉트, 라우팅 |

## 출처 / 참고

- [[workers|Browser Workers]]
- [[rendering-pipeline|브라우저 렌더링 파이프라인]]
- [[edge-computing|프론트엔드 Edge Computing]]
- 관련 위키: [[ipv4-vs-ipv6]]

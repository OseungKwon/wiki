---
title: libuv
aliases:
  - Node.js 이벤트 루프
  - Node.js 비동기 I/O
tags:
  - node-js
  - libuv
  - event-loop
  - async-io
created: 2026-03-30
updated: 2026-03-30
reviewed: false
---

## 정의

크로스 플랫폼 비동기 I/O 라이브러리. Node.js의 이벤트 루프와 논블로킹 I/O의 핵심 엔진으로, OS별 비동기 메커니즘(Linux epoll, macOS kqueue, Windows IOCP)을 추상화한다.

## 상세 설명

### 아키텍처

```
┌───────────────────────────────┐
│           Node.js             │
│  (V8 엔진 + JS 코드 실행)      │
├───────────────────────────────┤
│            libuv              │
│  ┌─────────────┬────────────┐ │
│  │  Event Loop  │Thread Pool │ │
│  │  (싱글스레드) │ (4개 기본)  │ │
│  └─────────────┴────────────┘ │
├───────────────────────────────┤
│    OS 커널 (epoll/kqueue/IOCP) │
└───────────────────────────────┘
```

- **V8**: JavaScript 파싱/컴파일/실행 (JIT) — "코드를 빠르게 실행"
- **libuv**: 비동기 I/O, 이벤트 루프, 스레드 풀 — "I/O를 논블로킹으로 처리"
- Node.js는 이 둘을 바인딩으로 연결한다

### Event Loop 페이즈

```
   ┌──────────┐
   │  timers   │ ← setTimeout, setInterval 콜백
   ├──────────┤
   │ pending   │ ← 시스템 에러 콜백 등
   ├──────────┤
   │  poll     │ ← I/O 콜백 (네트워크, 파일). 대부분 여기서 대기
   ├──────────┤
   │  check    │ ← setImmediate 콜백
   ├──────────┤
   │  close    │ ← socket.on('close') 등
   └──────────┘
   (각 페이즈 사이에 microtask 큐 실행: Promise, queueMicrotask)
```

### Event Loop vs Thread Pool

| Event Loop (싱글 스레드) | Thread Pool (기본 4개) |
|---|---|
| TCP/UDP 소켓 | 파일 시스템 (fs) |
| DNS 조회 (일부) | DNS 조회 (getaddrinfo) |
| 파이프, 시그널 | 압축 (zlib), 암호화 (crypto) |

Thread Pool 크기: `UV_THREADPOOL_SIZE` 환경 변수로 조절 (최대 1024).

### 주요 실무 포인트

**Event Loop 블로킹 주의**
```typescript
// Bad — 이벤트 루프를 막음
const result = heavyComputation();

// Good — Worker Thread로 위임
const { Worker } = require('worker_threads');
```

**setImmediate vs setTimeout(fn, 0)**
- `setTimeout(fn, 0)` → timers 페이즈 (최소 1ms 딜레이)
- `setImmediate(fn)` → check 페이즈 (poll 직후)
- I/O 콜백 안에서는 `setImmediate`가 항상 먼저 실행

**process.nextTick vs Promise**
- 둘 다 microtask이지만 `nextTick`이 Promise보다 우선
- `nextTick` 남용 시 I/O starvation 발생 가능

## 출처 / 참고

- [libuv 공식 문서](https://docs.libuv.org/)
- [Node.js 공식 - The Node.js Event Loop](https://nodejs.org/en/learn/asynchronous-work/event-loop-timers-and-nexttick)
- [[rendering-pipeline|브라우저 렌더링 파이프라인]] — 브라우저 측 이벤트 루프와 비교
- [[request-animation-frame]] — 브라우저의 프레임 기반 스케줄링
- [[libuv-thread-pool|libuv Thread Pool 심화]] — 스레드 풀 내부 구조, 포화 동작, 사이징 가이드

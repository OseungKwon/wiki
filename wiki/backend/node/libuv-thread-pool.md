---
title: libuv Thread Pool 심화
aliases:
  - UV_THREADPOOL_SIZE
  - Node.js 스레드 풀
  - Thread Pool
tags:
  - node-js
  - libuv
  - thread-pool
  - async-io
  - performance
  - concurrency
created: 2026-03-30
updated: 2026-03-30
reviewed: false
---

## 정의

**Thread Pool**은 미리 생성된 워커 스레드 집합을 재사용하여 작업을 처리하는 동시성 패턴이다. 요청마다 스레드를 생성/파괴하는 비용을 제거하고, 동시 스레드 수를 제어하여 시스템 리소스 고갈을 방지한다.

libuv의 Thread Pool은 이 패턴의 **고정 크기(fixed-size)** 구현체로, Node.js에서 OS 비동기 API가 없는 블로킹 작업(파일 I/O, DNS lookup, crypto, zlib)을 메인 스레드 밖에서 처리한다. 기본 4개, 최대 1024개.

## 상세 설명

### 스레드 풀이 필요한 이유

스레드 생성에는 비용이 크다:

| 항목 | 비용 |
|---|---|
| OS 스레드 생성 | ~20-100 us |
| 스레드 풀 태스크 제출 | ~0.1-1 us (큐 enqueue + signal) |
| 스레드당 스택 메모리 | Linux 8MB virtual / ~12KB RSS |
| 컨텍스트 스위칭 | 1-5 us (warm), 10-30 us (contention) |

요청마다 스레드를 만들면 1만 요청 = 1만 스레드 = **80GB 가상 메모리** + 스케줄러 과부하. 스레드 풀은 N개만 유지하면서 작업을 큐에 넣어 순차 처리하므로 비용을 상각(amortize)한다.

### 스레드 풀 패턴 종류

| 패턴 | 특징 | 대표 구현 |
|---|---|---|
| **Fixed-size** | N개 고정, 단순하고 예측 가능 | libuv, Java `newFixedThreadPool` |
| **Dynamic/Elastic** | core~max 범위에서 신축 | Java `ThreadPoolExecutor` |
| **Work-stealing** | 워커별 로컬 큐, 유휴 워커가 다른 큐에서 훔침 | Java ForkJoinPool, Go 스케줄러, Rust Tokio |
| **Thread-per-core** | 코어당 1스레드, 공유 없음 | ScyllaDB/Seastar |

libuv는 **Fixed-size + Slow I/O 스로틀링** 조합이다.

### 풀 사이징 공식 (Brian Goetz)

```
pool_size = N_cpu × U_cpu × (1 + W/C)

N_cpu = CPU 코어 수
U_cpu = 목표 CPU 사용률 (0~1)
W/C   = 대기시간/연산시간 비율
```

예: 8코어, I/O 대기 90%, CPU 사용률 100% → `8 × 1 × (1 + 9) = 80 스레드`

- **CPU-bound**: 코어 수 (또는 +1)
- **I/O-bound**: 대기 비율에 비례하여 크게
- **혼합**: CPU/IO 풀을 분리하여 기아 방지

### libuv 내부 아키텍처

첫 비동기 작업 요청 시 lazy 초기화. `uv_once()`로 한 번만 생성.

```
┌─────────────────────────────────────────────────┐
│                  Main Thread                     │
│              (Event Loop - 싱글 스레드)            │
│                                                  │
│  ┌──────────┐  ┌──────────┐  ┌───────────────┐  │
│  │  timers   │→│  pending  │→│  poll (epoll/  │  │
│  │           │  │ callbacks │  │  kqueue)      │  │
│  └──────────┘  └────▲─────┘  └───────────────┘  │
│                     │ uv_async_send()            │
├─────────────────────┼───────────────────────────┤
│              Thread Pool (기본 4개)               │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐   │
│  │Worker 1│ │Worker 2│ │Worker 3│ │Worker 4│   │
│  └────────┘ └────────┘ └────────┘ └────────┘   │
│         ▲         ▲         ▲         ▲         │
│         └─────────┴────┬────┴─────────┘         │
│                   Work Queue (무제한 FIFO)        │
└─────────────────────────────────────────────────┘
```

내부 3개 큐:

| 큐 | 용도 |
|---|---|
| `wq` | 일반 작업 대기열 |
| `slow_io_pending_wq` | 느린 I/O 전용 (fs, DNS) |
| `run_slow_work_message` | 느린 I/O 큐 확인 시그널 |

**기아 방지**: 느린 I/O는 최대 `max(1, pool_size / 2)` 스레드만 점유 가능.

### 워커 스레드 동작 흐름

```
워커 스레드 루프 (threadpool.c):
  1. mutex lock
  2. 작업 없으면 condition variable 대기
  3. wq에서 dequeue (없으면 slow_io_pending_wq 확인)
  4. mutex unlock
  5. work_cb 실행 (블로킹 syscall)
  6. 이벤트 루프 완료 큐에 결과 추가
  7. uv_async_send()로 이벤트 루프 깨움
```

### 작업 처리 전체 라이프사이클

```
JS: fs.readFile('data.txt', cb)
  → libuv: uv_fs_read() → uv_queue_work()
    → 워커 스레드: open() + read() (블로킹 syscall)
    → 완료 → 이벤트 루프 완료 큐에 추가
    → uv_async_send() → poll phase 깨움
  → 메인 스레드: pending callbacks에서 JS 콜백 실행
```

### 스레드 풀 사용 작업 vs OS 비동기 작업

**스레드 풀** (블로킹 → 워커 위임):

| 카테고리 | 작업 |
|---|---|
| 파일 시스템 | 모든 `fs.*` — open, read, write, stat 등 |
| DNS | `dns.lookup()` — `getaddrinfo(3)` (블로킹) |
| Crypto | `pbkdf2`, `scrypt`, `randomBytes` |
| Zlib | `deflate`, `inflate` |

**OS 비동기** (스레드 풀 미사용):

| 카테고리 | 메커니즘 |
|---|---|
| TCP/UDP 소켓 | epoll / kqueue / IOCP |
| `dns.resolve()` | c-ares (소켓 기반 비동기) |
| 타이머 | 이벤트 루프 min-heap |

**파일 I/O가 OS 비동기를 안 쓰는 이유**: Linux `epoll`은 일반 파일에서 항상 "ready" 반환 (디스크 I/O 완료와 무관). `O_NONBLOCK`도 무효. `io_uring`만이 진정한 비동기 파일 I/O이나 libuv에서 아직 실험적.

### 포화(Saturation) 시 동작

**백프레셔 없음.** 큐는 무제한 성장. 포화 = 에러가 아닌 레이턴시 증가.

```
시간 →
Thread 1: ████ pbkdf2 (200ms) ████
Thread 2: ████ pbkdf2 (200ms) ████
Thread 3: ████ pbkdf2 (200ms) ████
Thread 4: ████ pbkdf2 (200ms) ████
                                  ↑ fs.readFile 여기서야 시작
                                  ↑ dns.lookup 여기서야 시작
```

증상:
- `fs.stat()`: <1ms → 수백ms
- DNS 타임아웃 → HTTP 연결 실패
- Head-of-line blocking

### 일반적인 스레드 풀 함정

| 함정 | 설명 | libuv 해당 여부 |
|---|---|---|
| **풀 고갈** | 모든 스레드가 블로킹 작업 중, 신규 작업 무한 대기 | O — 무제한 큐 |
| **태스크→태스크 데드락** | 풀 내 태스크가 같은 풀에 서브태스크 제출 후 결과 대기 | △ — JS는 싱글스레드라 드뭄 |
| **우선순위 역전** | FIFO 큐라 긴급 작업이 뒤에 밀림 | O — 큐 분리 없음 |
| **ThreadLocal 오염** | 이전 작업의 상태가 다음 작업에 누출 | X — C 레벨 관리 |

### dns.lookup() vs dns.resolve() 함정

```javascript
// ❌ 스레드 풀 — 포화 시 병목
dns.lookup('example.com', callback);
http.get('http://example.com');  // 내부적으로 dns.lookup()

// ✅ c-ares — 스레드 풀 무관
dns.resolve4('example.com', callback);
```

### UV_THREADPOOL_SIZE 설정

```bash
UV_THREADPOOL_SIZE=16 node server.js  # 런타임 변경 불가
```

| 항목 | 값 |
|---|---|
| 기본 | 4 |
| 최소 | 1 |
| 최대 | 1024 (하드코딩) |
| Slow I/O 최대 동시 | `max(1, pool_size / 2)` |
| 스레드당 메모리 | ~2MB (스택) |

| 워크로드 | 권장 |
|---|---|
| 일반 웹 서버 | 4 (기본) |
| 파일 I/O 집중 | 16~64 |
| Crypto 헤비 | CPU 코어 × 2 |
| 극단적 I/O | 128 (스택 256MB 주의) |

## 출처 / 참고

- [libuv 공식 - Thread pool](https://docs.libuv.org/en/v1.x/threadpool.html)
- [libuv 소스 - threadpool.c](https://github.com/libuv/libuv/blob/v1.x/src/threadpool.c)
- [Node.js 공식 - Event Loop](https://nodejs.org/en/learn/asynchronous-work/event-loop-timers-and-nexttick)
- Brian Goetz, *Java Concurrency in Practice* — 풀 사이징 공식
- [[libuv]] — libuv 개요 및 이벤트 루프 전체 구조

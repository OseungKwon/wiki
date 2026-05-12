---
title: Browser Workers
aliases: [Web Worker, Service Worker, SharedWorker, 워커]
tags:
  - browser
  - web-worker
  - service-worker
  - pwa
  - performance
created: 2026-05-12
updated: 2026-05-12
reviewed: false
---

## 정의

브라우저가 메인 스레드와 별도로 실행하는 백그라운드 스크립트. DOM에 직접 접근할 수 없고, `postMessage`로 메인 스레드와 통신한다. 모두 `WorkerGlobalScope`를 공통 기반으로 상속한다.

## 상세 설명

### Worker 계층 구조

```
WorkerGlobalScope (공통 기반)
├── DedicatedWorkerGlobalScope  → Web Worker
├── SharedWorkerGlobalScope     → SharedWorker
└── ServiceWorkerGlobalScope    → Service Worker
```

공통 특성:
- 메인 스레드와 별도의 스레드에서 실행
- DOM 접근 불가, `window` 객체 없음 (`self` 사용)
- `postMessage`로 통신

### Web Worker

메인 스레드에서 무거운 연산을 분리하기 위한 **범용 백그라운드 스레드**.

```js
// main.js
const worker = new Worker('heavy.js');
worker.postMessage(largeData);
worker.onmessage = (e) => console.log(e.data);

// heavy.js
self.onmessage = (e) => {
  const result = heavyComputation(e.data);
  self.postMessage(result);
};
```

- 생성한 탭에서만 접근 가능
- 페이지와 함께 종료
- 페이지당 여러 개 생성 가능
- 활용: 대량 데이터 정렬/필터링, 이미지 처리, 암호화, CSV/Excel 파싱

### SharedWorker

같은 origin의 **여러 탭이 하나의 Worker를 공유**한다.

```js
const shared = new SharedWorker('shared.js');
shared.port.postMessage('hello');
```

- 탭 간 상태 동기화, WebSocket 연결을 하나로 유지할 때 유용

### Service Worker

네트워크 요청을 가로채는 **프록시 역할**의 특수 Worker. Web Worker 공통 기반 위에 네트워크 계층 API가 추가된 형태.

```js
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open('v1').then((cache) =>
      cache.addAll(['/index.html', '/style.css', '/app.js'])
    )
  );
});

self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((cached) =>
      cached || fetch(event.request)
    )
  );
});
```

#### 생명주기

1. **register** — `navigator.serviceWorker.register('/sw.js')`
2. **install** — 정적 자산 프리캐시
3. **activate** — 이전 버전 캐시 정리, 제어권 인수
4. **fetch/push/sync** — 실제 이벤트 처리

#### PWA의 핵심 기술인 이유

PWA가 네이티브 앱처럼 동작하려면 오프라인 동작, 설치 가능, 백그라운드 처리가 필요하다. Service Worker가 세 가지를 모두 가능하게 한다.

- **오프라인 동작**: fetch 이벤트를 가로채 캐시된 응답 반환
- **설치 가능**: 유효한 Service Worker가 등록되어 있어야 브라우저가 "홈 화면에 추가" 프롬프트를 띄움
- **백그라운드 처리**: 페이지가 닫혀도 Push Notification 수신, Background Sync 가능

#### 프로세스 레벨 동작

페이지를 닫아도 살아있을 수 있는 이유는 **브라우저 프로세스 레벨**에서 동작하기 때문이다.

```
Browser Process (브라우저 전체 관리)
├── Renderer Process (탭 단위)  ← 탭 닫으면 종료
├── GPU Process
├── Network Process
└── Service Worker Process      ← 탭과 독립, 브라우저가 관리
```

- 특정 탭에 종속되지 않고 **origin 단위로 등록**
- 항상 떠 있는 것은 아님 — 이벤트가 없으면 종료, 이벤트 발생 시 브라우저가 다시 깨움

#### Origin Scope

`프로토콜 + 호스트 + 포트` 단위로 종속. scope로 범위를 좁힐 수 있다.

```js
navigator.serviceWorker.register('/sw.js', { scope: '/app/' });
```

| | Session Storage | Service Worker |
|---|---|---|
| 단위 | origin + 탭 | origin |
| 탭 간 공유 | 불가 | 같은 origin 모든 탭이 공유 |
| 탭 닫으면 | 삭제됨 | 유지 |

cross-origin 공유는 Same-Origin Policy에 의해 불가능하다.

### 비교 요약

| | Web Worker | SharedWorker | Service Worker |
|---|---|---|---|
| 목적 | 연산 병렬화 | 탭 간 공유 | 네트워크 프록시 |
| 생명주기 | 페이지와 함께 | 마지막 탭 닫히면 종료 | 브라우저가 관리 |
| 개수 | 페이지당 여러 개 | origin당 하나 | origin당 하나 |
| HTTPS 필수 | 아니오 | 아니오 | 예 |

## 출처 / 참고

- [[rendering-pipeline|브라우저 렌더링 파이프라인]]
- [[network-request-flow|브라우저 네트워크 요청 흐름]]

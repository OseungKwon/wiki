---
title: Scheduler API
aliases: [postTask, scheduler.yield, isInputPending, requestIdleCallback]
tags:
  - javascript
  - web-api
  - performance
  - scheduling
created: 2026-03-26
updated: 2026-03-26
reviewed: false
---

## 정의

rAF의 "모든 작업이 동일 우선순위"라는 한계를 보완하는 브라우저 스케줄링 API들. 작업 우선순위 지정, 중간 양보, 유휴 시간 활용 등을 제공한다.

## 상세 설명

### scheduler.postTask (Chrome 94+)

우선순위 기반 작업 스케줄링. 세 가지 우선순위를 제공한다:

```js
// user-blocking: 즉각적인 사용자 반응 (가장 높음)
scheduler.postTask(() => updateButtonState(), { priority: 'user-blocking' });

// user-visible: 화면에 보이지만 즉각적이지 않아도 됨 (기본값)
scheduler.postTask(() => renderContent(), { priority: 'user-visible' });

// background: 사용자에게 보이지 않는 작업 (가장 낮음)
scheduler.postTask(() => prefetchData(), { priority: 'background' });
```

#### TaskController로 취소/우선순위 변경

```js
const controller = new TaskController({ priority: 'user-visible' });

const task = scheduler.postTask(() => heavyWork(), { signal: controller.signal });

controller.setPriority('user-blocking'); // 우선순위 상승
controller.abort(); // 작업 취소
```

### scheduler.yield (Chrome 129+)

긴 작업 중간에 브라우저에 제어권을 넘겨 Long Task를 방지한다:

```js
async function processItems(items) {
  for (const item of items) {
    processItem(item);
    await scheduler.yield(); // 브라우저에 양보 → 입력/렌더링 끼어들 수 있음
  }
}
```

### isInputPending (Chrome 87+)

사용자 입력이 대기 중인지 확인하여 양보 여부를 결정:

```js
function processQueue(queue) {
  while (queue.length > 0) {
    if (navigator.scheduling.isInputPending()) {
      requestAnimationFrame(() => processQueue(queue));
      return; // 입력에 양보
    }
    queue.shift()();
  }
}
```

### requestIdleCallback

브라우저가 여유 있을 때 실행. 렌더링 후 남은 시간에 동작한다:

```js
requestIdleCallback((deadline) => {
  while (deadline.timeRemaining() > 0 && tasks.length > 0) {
    tasks.shift()();
  }
  if (tasks.length > 0) requestIdleCallback(processRemaining);
}, { timeout: 2000 }); // 2초 안에는 반드시 실행
```

## 선택 가이드

```
즉각적인 DOM 변경 (같은 프레임)    → 동기 코드
빠른 상태 업데이트                  → queueMicrotask / Promise
시각적 업데이트 (프레임 동기화)      → requestAnimationFrame
우선순위가 필요한 작업              → scheduler.postTask
작업 중간 양보                     → scheduler.yield
여유 시간 배경 작업                → requestIdleCallback
```

### postTask vs rAF

| 특성 | rAF | postTask |
|------|-----|----------|
| 우선순위 | 단일 | 3단계 |
| vsync 동기화 | O | X |
| 취소/우선순위 변경 | cancelAnimationFrame만 | TaskController |
| 용도 | 시각적 업데이트 | 범용 작업 스케줄링 |

## 브라우저 지원 (2025 기준)

| API | Chrome | Firefox | Safari |
|-----|--------|---------|--------|
| postTask | 94+ | X | X |
| yield | 129+ | X | X |
| isInputPending | 87+ | X | X |
| requestIdleCallback | 47+ | 55+ | X |

### 폴백 패턴

```js
function scheduleWork(fn, priority = 'user-visible') {
  if ('scheduler' in globalThis && 'postTask' in scheduler) {
    return scheduler.postTask(fn, { priority });
  }
  switch (priority) {
    case 'user-blocking': return Promise.resolve().then(fn);
    case 'user-visible': return new Promise(r => requestAnimationFrame(() => r(fn())));
    case 'background': return new Promise(r =>
      (requestIdleCallback ?? setTimeout)(() => r(fn()))
    );
  }
}
```

## 출처 / 참고

- [MDN - Prioritized Task Scheduling](https://developer.mozilla.org/en-US/docs/Web/API/Prioritized_Task_Scheduling_API)
- [web.dev - Optimize long tasks](https://web.dev/articles/optimize-long-tasks)
- [[web-api/request-animation-frame|requestAnimationFrame 심화]]
- [[browser/rendering-pipeline|브라우저 렌더링 파이프라인]]

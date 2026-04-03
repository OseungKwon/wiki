---
title: Cancel+Reschedule 패턴
aliases: [rAF debounce, cancelAnimationFrame 패턴, 프레임 디바운스]
tags:
  - javascript
  - pattern
  - performance
  - debounce
  - request-animation-frame
created: 2026-03-26
updated: 2026-03-26
reviewed: false
---

## 정의

이전에 예약한 비동기 콜백을 취소하고 새로 예약하여, 연속 호출 시 마지막 요청만 실행되도록 하는 debounce 패턴. rAF 버전이 가장 대표적이다.

## 상세 설명

### 기본 구조

```js
let rafId = 0;

function onEvent() {
  cancelAnimationFrame(rafId);          // 이전 예약 취소
  rafId = requestAnimationFrame(() => { // 새로 예약
    // 처리
  });
}
```

### setTimeout debounce와의 관계

동일한 패턴의 변형이다:

```js
// setTimeout debounce (범용, 지연 시간 지정)
let timerId;
function onEvent() {
  clearTimeout(timerId);
  timerId = setTimeout(handler, delay);
}

// rAF debounce (시각적 업데이트 전용, 다음 vsync에 실행)
let rafId;
function onEvent() {
  cancelAnimationFrame(rafId);
  rafId = requestAnimationFrame(handler);
}
```

차이는 지연 시간이 고정값이 아니라 **다음 vsync**라는 점뿐이다.

### 플래그 가드와의 비교

```js
// 플래그 가드: 첫 호출 기준으로 다음 프레임에 실행
if (!rafId) {
  rafId = requestAnimationFrame(() => {
    rafId = 0;
    process();
  });
}

// cancel+reschedule: 마지막 호출 기준으로 다음 프레임에 실행
cancelAnimationFrame(rafId);
rafId = requestAnimationFrame(() => process());
```

**Set으로 수집하는 배칭 패턴**에서는 둘 다 같은 Set을 처리하므로 결과가 동일하다. 차이가 나는 경우는 **콜백이 호출 시점의 상태에 의존할 때**:

```js
// 스크롤 위치처럼 매 호출마다 값이 달라지는 경우
// cancel+reschedule이 최신 값을 보장
let rafId, lastY;
window.addEventListener('scroll', () => {
  lastY = window.scrollY;
  cancelAnimationFrame(rafId);
  rafId = requestAnimationFrame(() => {
    updateParallax(lastY); // 항상 마지막 스크롤 위치
  });
});
```

### 가드 없이 rAF만 호출하면?

```js
// 위험: rAF 콜백이 여러 개 큐에 쌓임
function onEvent() {
  requestAnimationFrame(() => {
    pendingAdded.forEach(processElement);
    pendingAdded.clear();
  });
}
```

이벤트 3회 → rAF 3개 등록 → 첫 번째가 처리+clear → 나머지 2개는 빈 Set 순회. 기능적 버그는 아니지만 불필요한 실행이 발생한다.

## 실무 적용

passorder tracking 패키지의 MutationObserver 배칭에서 이 패턴을 사용한다:

```js
const mutationObserver = new MutationObserver(mutations => {
  // Set에 요소 수집
  for (const mutation of mutations) { /* pendingAdded.add(...) */ }

  cancelAnimationFrame(mutationRafId);
  mutationRafId = requestAnimationFrame(() => {
    pendingRemoved.forEach(removeElement);
    pendingRemoved.clear();
    pendingAdded.forEach(processElement);
    pendingAdded.clear();
  });
});
```

이 경우 MutationObserver는 마이크로태스크, rAF는 렌더링 단계에서 실행되므로 모든 MO 콜백이 완료된 뒤 rAF가 한 번만 실행된다. cancel+reschedule과 플래그 가드 모두 결과가 동일하지만, cancel+reschedule이 더 널리 쓰이는 관용적 패턴이다.

## 출처 / 참고

- [[web-api/request-animation-frame|requestAnimationFrame 심화]]
- [[browser/rendering-pipeline|브라우저 렌더링 파이프라인]]
- [[web-api/mutation-observer|MutationObserver]]

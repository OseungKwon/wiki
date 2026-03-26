---
title: requestAnimationFrame 심화
aliases: [rAF, requestAnimationFrame, animation frame]
tags:
  - javascript
  - web-api
  - animation
  - performance
  - batching
created: 2026-03-26
updated: 2026-03-26
---

## 정의

브라우저의 다음 repaint 직전에 콜백을 실행하도록 예약하는 API. 모니터 주사율(vsync)에 동기화되어 시각적 업데이트의 정확한 타이밍을 보장한다.

## 상세 설명

### 기본 사용법

```js
const id = requestAnimationFrame((timestamp) => {
  // timestamp: DOMHighResTimeStamp (performance.now() 기준)
  element.style.transform = `translateX(${pos}px)`;
});
cancelAnimationFrame(id);
```

### 핵심 특성

- **1회성**: `setInterval`과 달리 한 번만 실행. 반복하려면 콜백 내에서 재귀 호출
- **vsync 동기화**: 모니터 주사율에 맞춤 (60Hz ≈ 16.67ms, 120Hz ≈ 8.33ms)
- **백그라운드 중단**: 탭이 비활성이면 호출 중단 → 배터리/CPU 절약
- **배칭**: 같은 프레임에 여러 rAF 등록 시 모두 같은 repaint 전에 실행

### 렌더링 파이프라인에서의 위치

```
매크로태스크 → 마이크로태스크(전부) → rAF → Style → Layout → Paint → Composite → rIC
```

rAF는 Style 계산 직전에 실행되므로, 여기서 DOM 변경하면 해당 프레임에 바로 반영된다. 자세한 파이프라인은 [[browser/rendering-pipeline|브라우저 렌더링 파이프라인]] 참고.

### 델타 타임 패턴

고주사율 모니터(120/144Hz)에서도 일정한 속도를 유지하려면 필수:

```js
let prev = 0;
function animate(timestamp) {
  const delta = timestamp - prev;
  prev = timestamp;
  pos += speed * (delta / 1000); // 초당 speed px
  requestAnimationFrame(animate);
}
```

## rAF 대신 다른 걸 쓰면 안 되는 이유

### vs setTimeout(fn, 0)

- vsync와 동기화 안 됨 — 프레임 사이 아무 때나 실행
- 중첩 5회 이상부터 최소 4ms 지연 강제 (HTML 스펙)
- 백그라운드 탭에서도 계속 실행 (1s 스로틀만 적용)

### vs queueMicrotask

- 큐가 빌 때까지 렌더링 차단 → 무한 루프 시 브라우저 프리즈
- DOM이 아직 안정화되지 않은 시점에 실행됨
- 콜백에서 레이아웃 읽기 시 forced reflow 발생 가능성 높음

### 비교표

| 특성 | setTimeout(0) | queueMicrotask | rAF |
|------|--------------|----------------|-----|
| 실행 위치 | 매크로태스크 | 마이크로태스크 | 렌더링 단계 |
| vsync 동기화 | X | X | O |
| 백그라운드 중단 | 부분적 (1s 스로틀) | X | O |
| 최소 지연 | 4ms (중첩 시) | 없음 | 1 vsync |
| 렌더링 차단 위험 | 낮음 | 높음 | 없음 |
| 타임스탬프 제공 | X | X | O (고정밀) |

## MutationObserver + rAF 배칭

[[web-api/mutation-observer|MutationObserver]] 콜백은 마이크로태스크로 실행된다. DOM이 아직 안정화되지 않은 시점이므로, rAF로 미뤄 안정된 DOM 상태에서 일괄 처리한다.

```
DOM 변경 1 → MO 콜백: Set에 수집, rAF 예약
DOM 변경 2 → MO 콜백: Set에 수집 (rAF 이미 예약됨)
DOM 변경 3 → MO 콜백: Set에 수집
...
다음 프레임 → rAF: Set을 순회하며 한 번에 처리
```

```js
class DomChangeBatcher {
  #pending = new Set();
  #rafId = null;

  constructor(target, callback) {
    this.callback = callback;
    this.observer = new MutationObserver((mutations) => {
      mutations.forEach(m => this.#pending.add(m.target));
      if (!this.#rafId) {
        this.#rafId = requestAnimationFrame(() => this.#flush());
      }
    });
    this.observer.observe(target, { childList: true, subtree: true });
  }

  #flush() {
    this.#rafId = null;
    this.callback(this.#pending);
    this.#pending.clear();
  }
}
```

**rAF에서 처리하는 이유:**
- 모든 DOM 변경이 끝난 후 실행됨 (한 프레임의 JS가 다 끝난 뒤)
- Paint 직전이므로 여기서 변경하면 해당 프레임에 반영
- Set으로 중복 제거 + 프레임당 1회 처리

## 고빈도 이벤트 배칭 (ticking 패턴)

스크롤/리사이즈 등 초당 100-200회 발생하는 이벤트를 프레임당 1회로 줄인다:

```js
let ticking = false;
let lastScrollY = 0;

window.addEventListener('scroll', () => {
  lastScrollY = window.scrollY;
  if (!ticking) {
    ticking = true;
    requestAnimationFrame(() => {
      // lastScrollY를 사용한 처리
      ticking = false;
    });
  }
}, { passive: true });
```

## Double rAF 패턴

rAF 안에서 rAF를 한 번 더 호출하여, 초기 상태와 목표 상태를 **다른 프레임**에서 설정하는 패턴.

### 필요한 경우

**CSS transition 시작**: 브라우저가 from/to 상태를 모두 알아야 transition이 작동한다. 같은 프레임에서 둘 다 설정하면 transition이 무시될 수 있다.

```js
element.style.opacity = '0';
requestAnimationFrame(() => {
  // 1프레임: opacity: 0이 커밋됨
  element.style.transition = 'opacity 0.3s';
  requestAnimationFrame(() => {
    // 2프레임: 이제 transition 시작 가능
    element.style.opacity = '1';
  });
});
```

**display: none → block 후 애니메이션**: `display: none` 요소는 레이아웃 트리에 없으므로, block으로 바꾼 뒤 한 프레임 기다려야 transition 적용 가능.

### 대안

- `void element.offsetHeight` — 강제 reflow로 브라우저에 스타일 계산 강제 (성능 비용)
- CSS `@starting-style` (Chrome 117+) — JS 없이 CSS만으로 해결

```css
@starting-style {
  .element { opacity: 0; }
}
.element {
  opacity: 1;
  transition: opacity 0.3s;
}
```

## 프레임워크 배칭과의 비교

| | React 18 | Vue 3 | rAF 배칭 |
|---|---|---|---|
| 메커니즘 | MessageChannel (매크로태스크) | Promise (마이크로태스크) | 렌더링 단계 |
| 속도 | <4ms | <1ms | ~16ms |
| 프레임 동기화 | X | X | O |
| 용도 | 상태 업데이트 | 상태 업데이트 | 시각적 DOM 조작 |

React는 초기에 rAF를 썼다가 MessageChannel로 전환했다. 이유:
- React의 상태 업데이트가 반드시 화면 갱신과 동기화될 필요 없음
- 백그라운드 탭에서도 상태 처리가 되어야 함
- MessageChannel은 4ms 지연 없이 매크로태스크 생성 가능

## Polyfill 역사

- **2011**: Firefox 4가 `mozRequestAnimationFrame`으로 최초 구현
- **2011-2013**: 벤더 접두사 시대 (`webkit-`, `moz-`, `ms-`, `o-`)
- **Paul Irish polyfill** (2011): `setTimeout`으로 16ms 시뮬레이션하는 유명한 snippet
- **2013**: 모든 주요 브라우저가 표준 지원
- **타임스탬프 진화**: 없음 → DOMTimeStamp → DOMHighResTimeStamp (마이크로초)
- **2018**: Spectre 취약점 이후 타임스탬프 정밀도 0.1ms로 제한

현재는 polyfill 불필요. `Cross-Origin-Isolated` 컨텍스트에서는 마이크로초 정밀도 복원 가능.

## 출처 / 참고

- [MDN - requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame)
- [HTML Spec - Animation Frames](https://html.spec.whatwg.org/multipage/imagebitmap-and-animations.html#animation-frames)
- [[browser/rendering-pipeline|브라우저 렌더링 파이프라인]]
- [[web-api/mutation-observer|MutationObserver]]
- [[browser/scheduler-api|Scheduler API]]

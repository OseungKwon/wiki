---
title: 브라우저 렌더링 파이프라인
aliases: [browser rendering, 렌더링 과정, critical rendering path]
tags:
  - browser
  - rendering
  - event-loop
  - microtask
  - performance
created: 2026-03-26
updated: 2026-03-26
---

## 정의

브라우저가 HTML, CSS, JS를 받아 화면에 픽셀을 출력하기까지의 전체 과정이다. 파싱 → 렌더 트리 → 레이아웃 → 페인트 → 컴포지팅의 파이프라인으로 구성되며, 이벤트 루프의 태스크/마이크로태스크 실행과 맞물려 동작한다.

## 상세 설명

### 1. 파싱 (Parsing)

#### HTML 파싱 → DOM 트리

```
Bytes → Characters → Tokens → Nodes → DOM Tree
```

- 토크나이저가 `<div>`, `</div>` 등을 토큰으로 분리
- 트리 빌더가 토큰을 노드로 변환하며 부모-자식 관계 구성
- 파싱은 점진적(incremental)으로 진행 — 전체 HTML을 기다리지 않음

#### CSS 파싱 → CSSOM 트리

```
Bytes → Characters → Tokens → Nodes → CSSOM Tree
```

- CSS는 **렌더 블로킹** 리소스 — CSSOM 완성 전까지 렌더 트리를 만들 수 없음
- 셀렉터는 **오른쪽에서 왼쪽**으로 매칭

#### JavaScript와 파싱 중단

```
HTML 파싱 중 <script> 만남
  → HTML 파싱 중단 (parser blocking)
  → CSSOM 완성 대기 (JS가 스타일 읽을 수 있으므로)
  → JS 다운로드 + 실행
  → HTML 파싱 재개
```

- `defer`: HTML 파싱 완료 후 실행, 순서 보장
- `async`: 다운로드 완료 즉시 실행, 순서 미보장
- `type="module"`: 기본이 `defer`와 동일

#### Speculative Parsing

메인 파서가 JS에 블로킹된 동안, **프리로드 스캐너(preload scanner)**가 나머지 HTML을 미리 스캔하여 리소스 다운로드를 선행 시작한다.

### 2. 렌더 트리 구축

```
DOM + CSSOM → Render Tree
```

- `display: none` → 렌더 트리에서 **제외**
- `visibility: hidden` → 렌더 트리에 **포함** (공간 차지)
- `<head>`, `<script>` 등 비시각적 요소 제외
- pseudo-element(`:before`, `:after`)는 DOM에 없지만 렌더 트리에 **포함**

### 3. 레이아웃 (Layout / Reflow)

각 노드의 정확한 위치와 크기를 계산한다.

- 뷰포트 기준 루트에서 하향식 계산
- `%`, `em`, `vw` 등 → 픽셀 절대값 변환
- **글로벌 레이아웃**: 전체 재계산 (초기 로드, 뷰포트 리사이즈)
- **인크리멘탈 레이아웃**: dirty 노드와 하위만 재계산

#### 강제 동기 레이아웃을 유발하는 읽기

```
offsetTop/Left/Width/Height, scrollTop/Left/Width/Height,
clientTop/Left/Width/Height, getComputedStyle(), getBoundingClientRect()
```

#### Layout Thrashing

```js
// BAD — 읽기/쓰기 반복 → 매번 강제 reflow
for (const el of elements) {
  el.style.width = container.offsetWidth + 'px';
}

// GOOD — 읽기 먼저, 쓰기 나중
const width = container.offsetWidth;
for (const el of elements) {
  el.style.width = width + 'px';
}
```

### 4. 페인트 (Paint)

레이아웃 정보를 기반으로 **페인트 레코드**(그리기 명령 목록)를 생성한다.

페인트 순서: background-color → background-image → border → children → outline → z-index stacking context

Repaint만 유발하는 속성: `color`, `background-color`, `visibility`, `box-shadow`, `border-radius`, `outline`

### 5. 컴포지팅 (Compositing)

#### 레이어 분리 조건

- `will-change: transform | opacity`
- `transform: translateZ(0)` / `translate3d()`
- `position: fixed`
- `<video>`, `<canvas>`, CSS `filter`, `mix-blend-mode`

#### 래스터라이제이션 → 합성

각 레이어를 타일 단위로 GPU에서 비트맵으로 변환 후, 순서대로 합성하여 최종 프레임을 출력한다.

**컴포지터만 사용하는 속성** (reflow/repaint 없이 가장 빠름): `transform`, `opacity` (별도 레이어일 때)

### 6. 파이프라인 재실행 범위

| 변경 | 트리거 범위 |
|------|------------|
| 기하학적 속성 (`width`, `margin`, `top`...) | Layout → Paint → Composite |
| 시각적 속성 (`color`, `background`...) | Paint → Composite |
| `transform`, `opacity` (레이어 분리 시) | **Composite만** |

## 이벤트 루프와 렌더링

### 이벤트 루프 한 사이클

```
1. [Task Queue]에서 매크로태스크 1개 실행
   (setTimeout, setInterval, I/O, UI events 등)

2. [Microtask Queue] 전부 비울 때까지 실행
   (Promise.then, queueMicrotask, MutationObserver, async/await 후속)
   ※ 실행 중 새 마이크로태스크가 추가되면 그것도 전부 실행

3. 렌더링 기회 판단 (~16.67ms = 60fps)
   └─ YES → rAF → Style → Layout → Paint → Composite → rIC
   └─ NO → 1번으로
```

### 마이크로태스크의 위치와 영향

마이크로태스크는 **렌더링 전에 전부** 실행된다:

```js
Promise.resolve().then(() => {
  document.body.style.background = 'red';
  Promise.resolve().then(() => {
    document.body.style.background = 'blue';
    // red는 화면에 절대 보이지 않음 — 최종값 blue만 렌더링
  });
});
```

- 마이크로태스크에서 DOM을 여러 번 변경해도 **최종 상태만** 렌더링됨
- 마이크로태스크가 무한히 쌓이면 **렌더링이 영원히 차단**됨 (UI 프리즈)

### requestAnimationFrame

rAF는 렌더링 직전에 실행되므로, 애니메이션 로직에 정확한 타이밍을 보장한다.

### MutationObserver

마이크로태스크로 동작하므로 DOM 변경 후, 렌더링 전에 실행된다. 콜백에서 추가 수정해도 최종값만 화면에 반영된다.

### 태스크 우선순위

```
[높음]  마이크로태스크        렌더링 전 전부 소진
  │    rAF 콜백             렌더링 직전 1회
  │    렌더링                브라우저 판단 (~60fps)
  │    매크로태스크           1 사이클에 1개
[낮음]  requestIdleCallback   렌더링 후 여유 시간에
```

### 한 프레임의 생애

```
매크로태스크 → 마이크로태스크(전부) → rAF → Style → Layout → Paint → Composite → rIC
```

## 출처 / 참고

- [MDN - Critical Rendering Path](https://developer.mozilla.org/en-US/docs/Web/Performance/Critical_rendering_path)
- [HTML Spec - Event Loop Processing Model](https://html.spec.whatwg.org/multipage/webappapis.html#event-loop-processing-model)
- [[javascript/proxy|Proxy]], [[web-api/mutation-observer|MutationObserver]]


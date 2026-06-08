---
title: Promise.resolve vs new Promise
aliases: [Promise 생성 방식, Promise 래핑]
tags:
  - javascript
  - promise
  - async
  - microtask
created: 2026-06-08
updated: 2026-06-08
---

## 정의
`new Promise`는 executor를 즉시 동기 실행하며 `resolve`/`reject` 핸들로 상태를
직접 제어해 **비동기 작업을 새로 시작/래핑**할 때 쓴다. `Promise.resolve(value)`는
**이미 가진 값을 즉시 fulfilled Promise로 포장**하는 단축 함수다.

## 상세 설명

### 핵심 차이
| | `new Promise` | `Promise.resolve` |
|---|---|---|
| 역할 | 비동기 작업을 새로 시작/래핑 | 이미 있는 값을 Promise로 포장 |
| executor | 생성 즉시 동기 실행 | 없음 |
| 상태 제어 | `resolve`/`reject` 직접 호출 | 항상 즉시 fulfilled |
| 인스턴스 | 항상 새 인스턴스 생성 | thenable이면 그 상태를 따라감 |

### new Promise — 비동기 작업을 직접 만든다
콜백·이벤트·타이머처럼 "언제 끝날지 모르는" 작업을 Promise로 감쌀 때 사용한다.
executor는 `resolve`/`reject`라는 상태 전환 핸들을 받아 원하는 타이밍에 성공/실패를
결정한다.

```js
function delay(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

function loadImage(src) {
  return new Promise((resolve, reject) => {
    const img = new Image();
    img.onload = () => resolve(img);
    img.onerror = reject;
    img.src = src;
  });
}
```

> 안티패턴: `fetch`/`axios`처럼 이미 Promise를 반환하는 API를 다시 `new Promise`로
> 감싸지 말 것 (explicit construction antipattern). 그냥 그 Promise를 return한다.

### Promise.resolve — 이미 있는 값을 감싼다
값이 이미 손에 있고 Promise 껍데기만 필요할 때 쓴다.

```js
Promise.resolve(42);
// 아래와 동일하지만 간결
new Promise((resolve) => resolve(42));
```

**thenable이면 따라간다**: Promise/thenable을 넣으면 이중 래핑하지 않고 그 상태를
그대로 따라간다. 이미 네이티브 Promise면 동일 인스턴스를 반환한다.

```js
Promise.resolve(p) === p; // p가 네이티브 Promise일 때 true
```

반면 `new Promise(r => r(...))`는 항상 새 인스턴스를 만든다.

## 실무 적용

**(1) 반환 타입 통일** — 캐시 히트(동기)/미스(비동기)를 호출부에서 동일하게 await:
```js
function getUser(identifier) {
  if (cache.has(identifier)) return Promise.resolve(cache.get(identifier));
  return fetch(`/users/${identifier}`).then((res) => res.json());
}
```

**(2) 체인 시작점** — 배열 순차 실행:
```js
tasks.reduce((chain, task) => chain.then(() => task()), Promise.resolve());
```

**(3) 마이크로태스크로 미루기**:
```js
Promise.resolve().then(() => { /* 현재 동기 코드 직후 실행 */ });
```

> `async` 함수는 반환값을 항상 `Promise.resolve`처럼 자동으로 감싸기 때문에,
> 요즘은 명시적 `Promise.resolve` 사용 빈도가 줄었다. 단 타입 통일·체인 시작
> 패턴에서는 여전히 유용하다. 실패 버전 `Promise.reject(error)`도 같은 맥락.

## 출처 / 참고
- MDN: Promise.resolve(), Promise() constructor
- [[브라우저 렌더링 파이프라인]] — 마이크로태스크 실행 시점

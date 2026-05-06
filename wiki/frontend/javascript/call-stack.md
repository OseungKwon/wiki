---
title: "Call Stack과 Stack Trace"
aliases:
  - 콜 스택
  - 스택 트레이스
  - error.stack
tags:
  - javascript
  - call-stack
  - stack-trace
  - debugging
  - error-handling
created: 2026-05-06
updated: 2026-05-06
reviewed: false
---

## 정의

Call Stack은 자바스크립트 엔진이 함수 호출 순서를 추적하는 LIFO(후입선출) 자료구조다. 에러 발생 시 엔진이 현재 콜 스택을 스냅샷으로 캡처한 것이 Stack Trace이며, `error.stack` 프로퍼티로 접근할 수 있다.

## 상세 설명

### 콜 스택 동작 원리

함수가 호출될 때마다 **스택 프레임(stack frame)**이 쌓이고, 함수가 반환되면 제거된다.

```js
function getUser(id) {
  return fetchData(`/users/${id}`);  // ③ 여기서 에러 발생
}

function processOrder(orderId) {
  const user = getUser(123);          // ② getUser 호출
  return { user, orderId };
}

function handleClick() {
  processOrder(456);                  // ① processOrder 호출
}

handleClick();
```

실행 시 콜 스택:

```
┌─────────────────────┐
│ fetchData()         │  ← 가장 나중에 호출 (맨 위)
├─────────────────────┤
│ getUser()           │
├─────────────────────┤
│ processOrder()      │
├─────────────────────┤
│ handleClick()       │
├─────────────────────┤
│ (global)            │  ← 가장 처음 (맨 아래)
└─────────────────────┘
```

### Stack Trace 캡처 — error.stack

에러가 발생하거나 `new Error()`가 호출되는 순간, JS 엔진이 **현재 콜 스택을 순회(walk)**하여 각 프레임에서 파일명, 함수명, 줄번호, 열번호를 추출하고 문자열로 조합한다.

```js
try {
  getUser(123);
} catch (e) {
  console.log(e.stack);
}
```

```
TypeError: Cannot read property 'name' of undefined
    at fetchData (app.js:12:5)        ← 에러 발생 지점
    at getUser (app.js:8:10)          ← 이 함수가 호출했고
    at processOrder (app.js:4:15)     ← 이 함수가 호출했고
    at handleClick (app.js:1:3)       ← 최초 호출 지점
```

가장 위가 에러 발생 지점, 가장 아래가 최초 호출 지점이다.

### 프로덕션에서의 문제: Minification

빌드 후 프로덕션 코드는 변수명·함수명이 축약되어 스택 트레이스를 읽을 수 없다:

```
// 원본: function processOrder(orderId) { ... }
// minify 후: function a(b){...}

TypeError: Cannot read property 'name' of undefined
    at a (app.min.js:1:4021)     ← 뭔지 알 수 없음
    at b (app.min.js:1:3845)
```

### Source Map으로 원본 매핑

Source Map(`.map` 파일)은 minify된 코드의 위치를 원본 코드 위치로 되돌리는 매핑 파일이다.

```json
{
  "version": 3,
  "file": "app.min.js",
  "sources": ["src/api/fetchData.ts", "src/order/process.ts"],
  "mappings": "AAAA,SAAS,EAAE,..."
}
```

`mappings` 필드가 **"minify된 파일의 줄:열 → 원본 파일의 줄:열"** 매핑을 Base64 VLQ로 인코딩한 것이다.

[[sentry|Sentry]] 같은 도구에서의 역매핑 흐름:

```
① 빌드: 원본 코드 → 번들러 → app.min.js + app.min.js.map
② 배포 전: .map 파일을 Sentry 서버에 업로드 (debug ID로 매칭)
③ 런타임: 에러 발생 → minify된 error.stack을 Sentry로 전송
④ 서버: source map으로 역매핑 → 원본 파일:줄:함수로 표시
```

### 비동기 코드의 스택 추적 한계

`setTimeout`, `Promise` 등 비동기 호출은 콜 스택이 끊긴다:

```js
async function loadOrder() {
  const data = await fetch('/api/order');  // 여기서 에러 시
  return data.json();
}
```

```
TypeError: Failed to fetch
    at fetch (native)
    at loadOrder (app.js:2:22)
    // 여기서 끝 — 누가 loadOrder를 호출했는지 모름
```

이를 보완하는 방법:
- **Async Stack Trace**: 크롬 등 최신 브라우저가 `await` 경계를 넘어 추적 지원
- **Breadcrumbs**: [[sentry|Sentry]]가 에러 발생 전 행동을 별도 타임라인으로 기록

### V8 엔진의 Stack Trace API

V8(Chrome/Node.js)은 `Error.captureStackTrace()`와 `Error.stackTraceLimit`을 제공한다:

```js
Error.stackTraceLimit = 20;  // 기본 10 → 20프레임으로 확장

// 커스텀 에러에 스택 트레이스 부착
function MyError(message) {
  Error.captureStackTrace(this, MyError);
  this.message = message;
}
```

`captureStackTrace`는 구조화된 `CallSite` 객체 배열로 스택을 제공하여, 프로그래밍적으로 파일명·줄번호·함수명 등을 개별 접근할 수 있다.

## 출처 / 참고

- [Error.prototype.stack - MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error/stack)
- [Error.captureStackTrace() - MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error/captureStackTrace)
- [Stack Trace API - V8](https://v8.dev/docs/stack-trace-api)
- [JavaScript Errors and Stack Traces in Depth](https://lucasfcosta.com/2017/02/17/JavaScript-Errors-and-Stack-Traces.html)
- [[sentry|Sentry - 프론트엔드 에러 모니터링]]

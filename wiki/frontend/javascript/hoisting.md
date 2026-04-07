---
title: "호이스팅(Hoisting)"
aliases: [hoisting, temporal-dead-zone, TDZ]
tags:
  - javascript
  - hoisting
  - tdz
  - scope
created: 2026-04-07
updated: 2026-04-07
reviewed: false
---

## 정의

**호이스팅(Hoisting)** 은 JavaScript 엔진이 코드 실행 전 컴파일 단계에서 선언을 스코프의 메모리에 먼저 등록하는 메커니즘이다. 코드가 물리적으로 이동하는 것이 아니라, 선언이 먼저 처리되어 최상단에 끌어올려진 것처럼 동작한다.

**TDZ(Temporal Dead Zone)** 는 변수가 호이스팅되어 스코프에 등록되었지만, 선언문이 실행되기 전까지 접근할 수 없는 구간이다. "Temporal"은 코드 위치가 아닌 실행 시간 기준임을 의미한다.

## 상세 설명

### 호이스팅 동작

```javascript
console.log(a); // undefined (에러 아님)
var a = 10;

// 엔진의 처리:
// var a;            ← 선언만 끌어올림
// console.log(a);   ← undefined
// a = 10;           ← 할당은 그 자리에
```

### 선언 방식별 차이

| 선언 | 호이스팅 | 초기화 | TDZ |
|------|---------|--------|-----|
| `var` | O | `undefined`로 | X |
| `let`/`const` | O | 안 됨 | O |
| `function` 선언 | O | 함수 전체 | X |
| `function` 표현식 | 변수 규칙 따름 | - | - |
| `class` | O | 안 됨 | O |

### 함수 선언 vs 표현식

```javascript
// 함수 선언 — 전체가 호이스팅
sayHi(); // ✅ 동작
function sayHi() { console.log('hi'); }

// 함수 표현식 — 변수 규칙을 따름
sayBye(); // ❌ TypeError (var) 또는 ReferenceError (let/const)
const sayBye = function() { console.log('bye'); };
```

### TDZ의 동작

```javascript
// ── TDZ 시작 ──
console.log(x); // ❌ ReferenceError
console.log(y); // ❌ ReferenceError
// ── TDZ 구간 ──
let x = 10;     // ← x의 TDZ 끝
const y = 20;   // ← y의 TDZ 끝
console.log(x); // ✅ 10
```

### TDZ가 존재하는 이유

`var`의 문제를 해결하기 위해 도입되었다. 선언 전에 변수를 사용하는 것은 거의 항상 실수이므로, 조용히 `undefined`를 반환하는 대신 에러를 발생시켜 버그를 조기에 발견하도록 한다.

```javascript
// var — 실수인데 조용히 넘어감
console.log(a); // undefined
var a = 5;

// let — 즉시 에러로 버그 조기 발견
console.log(b); // ReferenceError!
let b = 5;
```

### typeof도 TDZ에서 예외 없음

```javascript
// 아예 선언 안 된 변수 — typeof는 안전
console.log(typeof undeclared); // "undefined"

// TDZ에 있는 변수 — typeof도 에러
{
  console.log(typeof x); // ❌ ReferenceError
  let x = 2; // 이 블록의 x가 호이스팅되어 TDZ 형성
}
```

### let/const 호이스팅 증명

`let`/`const`가 호이스팅되지 않는다면, 블록 내에서 외부 변수를 참조할 수 있어야 한다. 하지만 TDZ 에러가 발생하는 것은 블록 내 선언이 호이스팅되어 외부 변수를 가린(shadow) 결과다.

```javascript
let x = 1;
{
  console.log(x); // ❌ ReferenceError (호이스팅 안 되면 1이 출력되어야 함)
  let x = 2;
}
```

## 출처 / 참고

- [MDN - Hoisting](https://developer.mozilla.org/en-US/docs/Glossary/Hoisting)
- [MDN - Temporal Dead Zone](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#temporal_dead_zone_tdz)

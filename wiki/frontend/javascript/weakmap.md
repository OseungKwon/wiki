---
title: WeakMap
aliases: [weak map, 위크맵]
tags:
  - javascript
  - weakmap
  - memory-management
  - data-structure
created: 2026-03-26
updated: 2026-03-26
reviewed: false
---

## 정의

`WeakMap`은 **키가 반드시 객체**여야 하며, 키에 대한 참조가 **약한 참조(weak reference)**인 키-값 컬렉션이다. 키 객체에 대한 다른 참조가 사라지면 GC가 해당 엔트리를 자동으로 수거한다. ES2015(ES6)에서 도입되었다.

## 상세 설명

### Map과의 차이

| 특성 | Map | WeakMap |
|------|-----|---------|
| 키 타입 | 아무 값 | **객체만** |
| 참조 방식 | 강한 참조 | 약한 참조 |
| 이터러블 | O | **X** |
| size 프로퍼티 | O | **X** |
| GC 대상 | X | **O** |

### 사용 가능한 메서드

4개만 제공된다. GC 타이밍이 비결정적이므로 순회(`forEach`, `keys`, `values`, `entries`)는 불가능하다.

```js
weakMap.set(key, value)
weakMap.get(key)
weakMap.has(key)
weakMap.delete(key)
```

### 약한 참조의 의미

WeakMap이 키 객체를 참조하더라도 GC 관점에서는 "참조 없음"과 동일하게 취급된다. 따라서 키 객체를 가리키는 다른 참조가 모두 사라지면, WeakMap 내 해당 엔트리도 자동으로 메모리에서 해제된다.

## 실무 적용

### 1. 프라이빗 데이터

```js
const privateData = new WeakMap();

class User {
  constructor(name) {
    privateData.set(this, { name });
  }
  getName() {
    return privateData.get(this).name;
  }
}
```

인스턴스가 GC되면 프라이빗 데이터도 자동 정리된다. ES2022의 `#` private 필드 등장 전에 널리 사용된 패턴이다.

### 2. DOM 노드에 메타데이터 연결

```js
const metadata = new WeakMap();

function track(element) {
  metadata.set(element, { clickCount: 0 });
}
// element가 DOM에서 제거되고 참조가 사라지면 메타데이터도 자동 해제
```

### 3. 객체 기반 캐싱 (메모이제이션)

```js
const cache = new WeakMap();

function heavyCompute(obj) {
  if (cache.has(obj)) return cache.get(obj);
  const result = expensiveOperation(obj);
  cache.set(obj, result);
  return result;
}
// obj 참조가 사라지면 캐시도 자동 정리 → 메모리 누수 없음
```

## 핵심 요약

- **최대 장점**: 메모리 누수 방지. 객체에 부가 데이터를 붙이되 생명주기를 방해하지 않는다.
- **제약**: 순회/열거 불가. 전체 항목을 조회해야 하면 `Map`을 사용한다.
- **관련**: [[javascript/proxy|Proxy]]와 함께 메타프로그래밍에 자주 활용된다.

## 출처 / 참고

- [MDN - WeakMap](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/WeakMap)

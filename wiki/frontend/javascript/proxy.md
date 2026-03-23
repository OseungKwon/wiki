---
title: JavaScript Proxy
aliases:
  - Proxy
  - JS Proxy
  - Reflect
tags:
  - javascript
  - proxy
  - metaprogramming
created: 2026-03-18
updated: 2026-03-18
---

## 정의

Proxy는 객체의 기본 동작(읽기, 쓰기, 삭제 등)을 가로채서 커스텀 로직을 주입할 수 있는 JavaScript 내장 메타프로그래밍 도구다. ES2015(ES6)에서 도입되었으며, 2016년 9월부터 모든 주요 브라우저에서 지원된다.

## 상세 설명

### 기본 구조

```javascript
const proxy = new Proxy(target, handler);
```

- **target**: 원본 객체. Proxy가 감싸는 대상
- **handler**: 트랩(trap)을 정의하는 객체. 어떤 동작을 가로챌지 결정
- **trap**: handler 내부의 메서드. JavaScript 엔진의 내부 메서드와 1:1로 대응

handler가 비어있으면 모든 동작이 원본 객체로 그대로 통과한다.

### 트랩 전체 목록 (13개)

| 트랩 | 가로채는 동작 | 트리거 예시 |
|------|-------------|-----------|
| `get(target, prop, receiver)` | 속성 읽기 | `proxy.name` |
| `set(target, prop, value, receiver)` | 속성 쓰기 | `proxy.name = 'x'` |
| `has(target, prop)` | 존재 여부 확인 | `'name' in proxy` |
| `deleteProperty(target, prop)` | 속성 삭제 | `delete proxy.name` |
| `ownKeys(target)` | 키 목록 조회 | `Object.keys(proxy)` |
| `apply(target, thisArg, args)` | 함수 호출 | `proxy()` |
| `construct(target, args)` | 생성자 호출 | `new proxy()` |
| `getPrototypeOf(target)` | 프로토타입 조회 | `Object.getPrototypeOf(proxy)` |
| `setPrototypeOf(target, proto)` | 프로토타입 설정 | `Object.setPrototypeOf(proxy, proto)` |
| `isExtensible(target)` | 확장 가능 여부 | `Object.isExtensible(proxy)` |
| `preventExtensions(target)` | 확장 방지 | `Object.preventExtensions(proxy)` |
| `getOwnPropertyDescriptor(target, prop)` | 속성 설명자 조회 | `Object.getOwnPropertyDescriptor(proxy, 'x')` |
| `defineProperty(target, prop, desc)` | 속성 정의 | `Object.defineProperty(proxy, 'x', {...})` |

> `apply`와 `construct`는 target이 함수일 때만 사용 가능하다.

### 핵심 트랩: get / set

실무에서 가장 많이 사용되는 두 트랩이다. 반응형 시스템(Vue, MobX, Valtio)의 핵심 메커니즘이기도 하다.

**get — 속성 읽기 가로채기:**

```javascript
const handler = {
  get(target, prop, receiver) {
    if (prop in target) return target[prop];
    return `"${prop}"은 존재하지 않습니다.`;
  }
};
const proxy = new Proxy({ name: '홍길동' }, handler);
proxy.name;    // '홍길동'
proxy.address; // '"address"은 존재하지 않습니다.'
```

**set — 속성 쓰기 가로채기 (유효성 검사):**

```javascript
const handler = {
  set(target, prop, value) {
    if (prop === 'age' && (typeof value !== 'number' || value < 0)) {
      throw new Error('나이는 0 이상의 숫자여야 합니다.');
    }
    target[prop] = value;
    return true; // 반드시 boolean 반환
  }
};
```

### Reflect — 트랩의 짝꿍

Reflect는 Proxy 트랩과 동일한 이름의 메서드를 제공하는 내장 객체로, 원래 동작을 안전하게 위임하는 역할을 한다.

**왜 `target[prop]` 대신 `Reflect.get()`을 써야 하는가?**

직접 접근은 상속 체인에서 `this` 바인딩 문제를 일으킨다:

```javascript
const parent = {
  _name: '부모',
  get name() { return this._name; }
};

// ❌ target[prop] — this가 parent에 고정됨
const bad = new Proxy(parent, {
  get(target, prop, receiver) { return target[prop]; }
});
const child = Object.create(bad);
child._name = '자식';
child.name; // '부모' (잘못된 결과)

// ✅ Reflect.get — receiver를 통해 올바른 this 전달
const good = new Proxy(parent, {
  get(target, prop, receiver) { return Reflect.get(target, prop, receiver); }
});
const child2 = Object.create(good);
child2._name = '자식';
child2.name; // '자식' (정상)
```

> **권장 패턴**: 트랩에서 커스텀 로직 수행 후 `Reflect`로 원래 동작을 위임한다.

### Proxy.revocable() — 취소 가능한 프록시

나중에 프록시를 완전히 비활성화할 수 있다. 보안이나 임시 접근 제어에 유용하다.

```javascript
const { proxy, revoke } = Proxy.revocable(target, handler);
proxy.name; // 정상 작동
revoke();   // 프록시 비활성화
proxy.name; // TypeError 발생
```

### 주의사항

**프라이빗 필드(`#field`) 접근 불가:**
```javascript
class Secret {
  #value;
  get value() { return this.#value; }
}
const proxy = new Proxy(new Secret(), {});
proxy.value; // TypeError — #value는 프록시 객체에 존재하지 않음
// 해결: get 트랩에서 target[prop]으로 원본에 위임
```

**내장 객체(Map, Set, Date) 호환 문제:**
```javascript
const proxy = new Proxy(new Map(), {});
proxy.set('key', 'val'); // TypeError — 내부 슬롯 접근 불가
// 해결: 메서드를 target에 bind
get(target, prop) {
  const value = target[prop];
  return value instanceof Function ? value.bind(target) : value;
}
```

**성능 오버헤드**: 매 트랩 호출마다 추가 함수 실행이 발생한다. 대량 데이터 처리 시 주의가 필요하다.

## React에서의 Proxy 활용

React 자체는 Proxy를 사용하지 않지만, 상태 관리 라이브러리들이 Proxy를 핵심 메커니즘으로 활용하여 **Fine-Grained Reactivity**를 구현한다.

### 동작 원리

```
[상태 변경]                    [렌더링]
state.count++                 useSnapshot(state)
     ↓                             ↓
set 트랩 발동               get 트랩으로 접근 속성 추적
     ↓                             ↓
변경된 속성 기록            구독 목록에 컴포넌트 등록
     ↓                             ↓
해당 속성을 구독한 컴포넌트만 리렌더 ←─┘
```

`useState`와의 근본적 차이:
- **useState**: 상태 전체가 교체되면 해당 컴포넌트 리렌더
- **Proxy 기반**: 속성 단위로 추적하여 실제 사용한 속성이 변경될 때만 리렌더

### 대표 라이브러리

- [[valtio|Valtio]] — 최소 API로 Proxy 상태를 단순화. 직접 mutation + 자동 리렌더 최적화
- [[mobx|MobX]] — 클래스 기반 OOP + Proxy. computed, reaction 등 풍부한 반응형 프리미티브
- Vue 3의 `reactive()`도 동일한 Proxy 메커니즘 사용 (Vue 2의 `Object.defineProperty`에서 전환)

## 출처 / 참고

- [요즘IT - JavaScript Proxy 완전 가이드](https://yozm.wishket.com/magazine/detail/3659/)
- [MDN - Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)

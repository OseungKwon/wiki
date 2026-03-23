---
title: Valtio
aliases:
  - valtio
  - proxy state
tags:
  - react
  - state-management
  - proxy
  - valtio
created: 2026-03-18
updated: 2026-03-18
---

## 정의

Valtio는 [Poimandres](https://github.com/pmndrs) (Zustand, Jotai, React Three Fiber 등을 만든 오픈소스 그룹)에서 개발한 [[proxy|Proxy]] 기반 React 상태 관리 라이브러리다. MobX/Immer의 Proxy 패턴을 최소한의 API로 단순화하는 것이 목표이며, "객체를 변경하면 컴포넌트가 반응한다"는 철학을 따른다.

## 상세 설명

### 주요 특징

- `proxy()`, `useSnapshot()`, `subscribe()` 3개 API로 대부분 처리
- action/reducer 패턴 없이 직접 mutation으로 상태 변경
- 접근한 속성만 추적하여 자동 리렌더 최적화 (selector 불필요)
- React 18/19 Suspense, Concurrent Mode 호환
- Vanilla JS에서도 사용 가능 (`valtio/vanilla`)
- Redux DevTools 연동 지원

### 핵심 API

```javascript
import { proxy, useSnapshot, subscribe } from 'valtio'

// 1. proxy() — 내부적으로 Proxy의 get/set 트랩 설정
const state = proxy({ count: 0, text: 'hello' })

// 2. 상태 변경 — 일반 객체처럼 직접 수정
state.count++
state.text = 'world'

// 3. useSnapshot() — 읽은 속성만 추적, 불변 스냅샷 반환
function Counter() {
  const snap = useSnapshot(state)
  return <button onClick={() => ++state.count}>{snap.count}</button>
  // snap.text를 사용하지 않으므로 text가 변경돼도 리렌더 안 됨
}

// 4. subscribe() — 컴포넌트 외부에서 변경 감지
subscribe(state, () => console.log('변경됨:', state))
```

### 고급 기능

**ref() — 비추적 객체:**
```javascript
import { ref } from 'valtio'
const state = proxy({
  count: 0,
  dom: ref(document.body) // 프록시되지 않음
})
```

**computed (getter):**
```javascript
const state = proxy({
  count: 1,
  get doubled() { return this.count * 2 }
})
```

**부분 구독:**
```javascript
subscribe(state.arr, () => console.log('배열만 감지'))
```

### Zustand와의 비교

같은 Poimandres 팀이 만들었지만 접근 방식이 다르다:

| | Valtio | Zustand |
|--|--------|---------|
| 업데이트 방식 | mutable (`state.count++`) | immutable (`set(s => ({...}))`) |
| 변경 감지 | Proxy 트랩 자동 감지 | 참조 비교 |
| 리렌더 최적화 | 속성 접근 자동 추적 | selector 수동 작성 |
| 멘탈 모델 | MobX에 가까움 | Redux에 가까움 |

## 출처 / 참고

- [Valtio GitHub](https://github.com/pmndrs/valtio)
- [Valtio 공식 문서](https://valtio.dev)
- [[proxy|JavaScript Proxy]]

---
title: MobX
aliases:
  - mobx
  - MobX React
tags:
  - react
  - state-management
  - proxy
  - mobx
  - observable
created: 2026-03-18
updated: 2026-03-18
---

## 정의

MobX는 2015년부터 사용되어 온 검증된 반응형 상태 관리 라이브러리다. 클래스 기반 OOP 패턴과 [[proxy|Proxy]]를 결합하여 자동 의존성 추적을 구현하며, MobX 5부터 Proxy를 기본 메커니즘으로 채택했다.

## 상세 설명

### 주요 특징

- `makeAutoObservable`로 클래스를 자동으로 Observable(Proxy)로 변환
- `observer()` HOC가 렌더링 중 접근한 속성을 자동 추적
- computed 값 자동 캐싱, reaction, autorun 등 풍부한 반응형 프리미티브
- 대규모 앱에서의 구조화에 강점 (도메인 모델링)
- Proxy 미지원 환경을 위한 ES5 fallback 제공

### 핵심 사용법

```javascript
import { makeAutoObservable } from 'mobx'
import { observer } from 'mobx-react-lite'

class CounterStore {
  count = 0

  constructor() {
    makeAutoObservable(this) // 모든 필드를 observable, 메서드를 action으로 변환
  }

  increment() { this.count++ }

  get doubled() { return this.count * 2 } // computed — 자동 캐싱
}

const store = new CounterStore()

const Counter = observer(() => (
  // get 트랩으로 count 접근 감지 → count 변경 시에만 리렌더
  <button onClick={() => store.increment()}>{store.count}</button>
))
```

### 핵심 개념

| 개념 | 역할 | 예시 |
|------|------|------|
| **observable** | Proxy로 감싼 추적 가능한 상태 | `count = 0` |
| **action** | 상태를 변경하는 메서드 | `increment()` |
| **computed** | 의존값이 변경될 때만 재계산되는 파생 값 | `get doubled()` |
| **reaction** | 상태 변경 시 실행되는 사이드 이펙트 | `autorun(() => ...)` |
| **observer** | 컴포넌트를 반응형으로 만드는 HOC | `observer(() => ...)` |

### Valtio와의 비교

| | MobX | Valtio |
|--|------|--------|
| 스타일 | 클래스 기반 OOP | 순수 객체 기반 |
| API 표면적 | 넓음 (observable, action, computed, reaction...) | 좁음 (proxy, useSnapshot, subscribe) |
| 구조화 | 도메인 모델 패턴에 적합 | 간단한 상태에 적합 |
| 러닝커브 | 중간 | 낮음 |

## 출처 / 참고

- [MobX 공식 문서](https://mobx.js.org)
- [Proxy State Management: MobX vs Valtio](https://www.frontendundefined.com/posts/monthly/proxy-state-management-mobx-valtio/)
- [[proxy|JavaScript Proxy]]
- [[valtio|Valtio]]

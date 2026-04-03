---
title: React Fiber WIP 트리
aliases:
  - Work-In-Progress Tree
  - WIP Tree
  - 더블 버퍼링
tags:
  - react
  - fiber
  - reconciliation
  - concurrent
created: 2026-04-03
updated: 2026-04-03
---

## 정의
React Fiber 아키텍처에서 사용하는 **더블 버퍼링** 기법으로, 현재 화면 상태를 나타내는 Current 트리와 다음 렌더링을 준비하는 WIP(Work-In-Progress) 트리 두 개를 유지하여 비동기적이고 중단 가능한 렌더링을 구현한다.

## 상세 설명

### 두 개의 Fiber 트리

| 트리 | 역할 |
|------|------|
| **Current 트리** | 현재 화면에 커밋된 상태를 반영 |
| **WIP 트리** | 다음 렌더링을 위해 변경 사항을 계산 중인 트리 |

각 Fiber 노드는 `alternate` 속성으로 상대 트리의 대응 노드를 참조한다.

```
[Current 트리] ←── 화면에 보이는 상태
     ↕ alternate
[WIP 트리]     ←── 백그라운드에서 작업 중
```

### 동작 흐름

1. **상태 변경 발생** → Current 트리를 기반으로 WIP 트리 작업 시작
2. **Render Phase** → WIP 트리를 순회하며 변경 사항 계산 (중단/재개 가능)
3. **Commit Phase** → 완성된 WIP 트리를 한 번에 DOM에 반영
4. **스왑** → WIP 트리가 새로운 Current 트리가 되고, 이전 Current는 다음 업데이트의 WIP로 재활용

### 더블 버퍼링을 쓰는 이유

- **일관성**: 작업이 완전히 끝난 후에만 화면 업데이트 → 중간 상태(tearing)가 사용자에게 노출되지 않음
- **중단 가능**: Render Phase에서 WIP 작업을 중단해도 Current 트리(화면)에 영향 없음
- **메모리 효율**: 매번 트리를 새로 생성하지 않고 두 트리를 번갈아 재활용
- **Concurrent Mode 지원**: 우선순위 높은 작업이 들어오면 진행 중인 WIP를 버리고 새로 시작 가능

## 출처 / 참고
- [React Source - ReactFiber.js](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiber.js)
- [[suspense-version-diff|React Suspense 버전별 동작 차이]]

---
title: React 렌더링 생애주기 — Fiber, WIP, Suspense 통합 정리
aliases:
  - React Rendering Lifecycle
  - Fiber Rendering Lifecycle
  - Suspense 내부 동작
tags:
  - react
  - fiber
  - suspense
  - concurrent
  - reconciliation
  - rendering
created: 2026-04-03
updated: 2026-04-03
reviewed: false
---

## 정의
React의 렌더링은 **Trigger → Render → Commit** 세 단계로 이루어진다. Fiber 아키텍처는 이 과정을 WIP 트리 위에서 수행하며, Suspense는 Render Phase에서 Promise를 throw하여 렌더링을 일시 중단하는 메커니즘이다.

## 상세 설명

### 전체 흐름

```
setState / props 변경
    │
    ▼
┌─────────────────────────────────────────────┐
│  1. Trigger Phase                           │
│  - 업데이트를 Lane(우선순위)에 등록          │
│  - Scheduler가 작업을 예약                   │
└─────────────────┬───────────────────────────┘
                  ▼
┌─────────────────────────────────────────────┐
│  2. Render Phase (중단/재개 가능)            │
│  - WIP 트리를 순회하며 변경 사항 계산        │
│  - beginWork: 위→아래 (자식 생성/재사용)     │
│  - completeWork: 아래→위 (effect 수집)      │
│  - Suspense: Promise throw 시 중단          │
└─────────────────┬───────────────────────────┘
                  ▼
┌─────────────────────────────────────────────┐
│  3. Commit Phase (동기적, 중단 불가)         │
│  - DOM 변경 일괄 적용                        │
│  - WIP 트리 → Current 트리로 스왑           │
│  - useLayoutEffect → 브라우저 paint          │
│  - → useEffect                              │
└─────────────────────────────────────────────┘
```

### Render Phase 상세: beginWork와 completeWork

Render Phase는 Fiber 트리를 DFS로 순회하며 두 함수를 반복한다.

**beginWork (위 → 아래)**
- Current Fiber와 WIP Fiber를 비교
- props/state가 변경되었으면 컴포넌트를 실행하여 자식 Fiber 생성
- 변경 없으면 기존 Fiber를 `bailout` (재사용)
- 자식이 있으면 자식으로 이동, 없으면 completeWork로 전환

**completeWork (아래 → 위)**
- DOM 노드 생성/업데이트 준비 (아직 실제 DOM에 반영하지 않음)
- side effect를 flags로 마킹 (Placement, Update, Deletion 등)
- sibling이 있으면 sibling의 beginWork로, 없으면 부모의 completeWork로 이동

```
       App (beginWork)
        │
        ▼
      Layout (beginWork)
        │
        ▼
      Header (beginWork → completeWork) → Sibling: Content
                                              │
                                              ▼
                                         Content (beginWork)
                                              │
                                              ▼
                                         Child (beginWork → completeWork)
                                              │
                                              ▲
                                         Content (completeWork)
        │
        ▲
      Layout (completeWork)
        │
        ▲
       App (completeWork)
```

### WIP 트리와 렌더링

[[fiber-wip-tree|Fiber WIP 트리]]의 더블 버퍼링이 이 과정에서 핵심 역할을 한다:

1. **Render 시작**: Current 트리의 Fiber 노드를 `alternate`로 복제하여 WIP 노드 생성
2. **Render 진행**: WIP 트리 위에서 beginWork/completeWork 수행
3. **Render 완료**: WIP 트리에 모든 변경 사항과 effect flags가 기록됨
4. **Commit**: WIP 트리의 effect를 DOM에 반영 후 `root.current = wipTree`로 스왑

이 분리 덕분에 Render Phase에서 에러가 발생하거나 Suspense로 중단되어도 **화면(Current 트리)은 안전하게 유지**된다.

### Suspense가 렌더링을 중단하는 메커니즘

Suspense는 **Promise를 throw하는 패턴**으로 동작한다.

```tsx
function UserProfile({ userPromise }) {
  const user = use(userPromise); // 미완료 시 Promise throw
  return <div>{user.name}</div>;
}
```

#### 중단 흐름

```
beginWork(Suspense boundary)
  │
  ▼
beginWork(UserProfile)
  → use(userPromise) 호출
  → Promise 미완료 → throw Promise
  │
  ▼
React가 throw를 catch
  │
  ├─ 가장 가까운 Suspense boundary를 찾음
  ├─ 해당 boundary의 fallback을 렌더링
  ├─ throw된 Promise에 then() 등록
  │
  ▼
Promise resolve 시
  → Suspense boundary부터 re-render 트리거
  → 이번엔 use()가 resolved value 반환
  → 정상 렌더링 완료
```

#### WIP 트리 관점에서의 Suspense

| 단계 | WIP 트리 상태 |
|------|--------------|
| 최초 렌더링 | Suspense boundary 아래에 fallback 서브트리를 WIP에 기록 |
| Promise throw | primary 서브트리 작업을 중단, fallback으로 전환 |
| Promise resolve | re-render 시 primary 서브트리를 WIP에 다시 구축 |
| Commit | fallback → primary 전환이 DOM에 반영 |

### Concurrent Rendering과 우선순위

React 18+에서 Scheduler는 **Lane** 시스템으로 업데이트 우선순위를 관리한다.

| Lane | 예시 | 특성 |
|------|------|------|
| SyncLane | `flushSync()` | 즉시 실행, 중단 불가 |
| InputContinuousLane | `onChange`, `onScroll` | 높은 우선순위, 연속 입력 |
| DefaultLane | `setState()` | 일반 업데이트 |
| TransitionLane | `startTransition()` | 낮은 우선순위, 중단 가능 |
| IdleLane | `useDeferredValue` | 유휴 시 처리 |

**Suspense + Transition 연동**:
```tsx
function SearchResults() {
  const [query, setQuery] = useState('');

  function handleChange(e) {
    // 입력은 SyncLane → 즉시 반영
    // 결과 렌더링은 TransitionLane → Suspense fallback 대신 이전 UI 유지
    startTransition(() => {
      setQuery(e.target.value);
    });
  }

  return (
    <>
      <input onChange={handleChange} />
      <Suspense fallback={<Skeleton />}>
        <Results query={query} />
      </Suspense>
    </>
  );
}
```

`startTransition` 안의 업데이트로 Suspense가 트리거되면, fallback을 **보여주지 않고** 이전 UI를 유지한다. 이는 WIP 트리에서 새 결과를 준비하면서 Current 트리(이전 UI)를 화면에 유지하는 더블 버퍼링의 장점이다.

### Commit Phase 상세

Commit Phase는 **동기적으로 실행**되며 세 단계로 나뉜다:

1. **Before Mutation**: `getSnapshotBeforeUpdate` 호출
2. **Mutation**: 실제 DOM 변경 (삽입, 수정, 삭제), `ref` 해제
3. **Layout**: `root.current = wipTree` 스왑, `ref` 재연결, `useLayoutEffect` 실행

이후 브라우저가 paint하고, 다음 microtask에서 `useEffect`가 실행된다.

## 출처 / 참고
- [React Source — ReactFiberWorkLoop.js](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberWorkLoop.js)
- [[fiber-wip-tree|React Fiber WIP 트리]] — 더블 버퍼링 구조
- [[suspense-version-diff|React Suspense 버전별 동작 차이]] — 버전별 API 변천
- [[rendering-pipeline|브라우저 렌더링 파이프라인]] — DOM 이후 브라우저 처리

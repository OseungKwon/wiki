---
title: React Suspense 버전별 동작 차이 (17 → 18 → 19)
aliases:
  - Suspense 버전 비교
  - React Suspense 변천사
tags:
  - react
  - suspense
  - concurrent
  - streaming-ssr
created: 2026-04-02
updated: 2026-04-02
---

## 정의

React Suspense는 비동기 작업(코드 스플리팅, 데이터 페칭 등)이 완료될 때까지 fallback UI를 보여주는 메커니즘이다. React 17, 18, 19를 거치며 지원 범위와 동작 방식이 크게 변화했다.

## 상세 설명

### React 17: 코드 스플리팅 전용

- `React.lazy()` + `<Suspense>`로 **코드 스플리팅만** 공식 지원
- Data fetching 미지원 (Relay 내부에서만 실험적 사용)
- SSR에서 Suspense 미지원 (`renderToString()`은 Suspense 무시)
- Concurrent Mode 없음 — 동기적 렌더링만 가능

### React 18: Concurrent + Streaming SSR

#### Concurrent Rendering
- `createRoot()`로 concurrent features 활성화
- 컴포넌트가 suspend되면 렌더링을 중단하고, 데이터 준비 시 재개

#### Streaming SSR
- `renderToPipeableStream()` 도입 — Suspense boundary 기준으로 HTML 점진적 스트리밍
- **Selective Hydration**: 사용자가 상호작용한 부분 우선 hydrate

#### Sibling 렌더링
- 하나가 suspend되면 **sibling도 렌더링 중단** → fallback 표시
- 여러 데이터를 병렬 fetch하려면 각각 별도 Suspense boundary 필요

```tsx
// React 18: A가 suspend → B는 렌더링 시도조차 안 됨
<Suspense fallback={<Loading />}>
  <ComponentA />
  <ComponentB />
</Suspense>
```

#### Transitions 연동
- `useTransition()` + Suspense로 이전 UI 유지하면서 백그라운드에서 새 UI 준비

### React 19: 병렬 Sibling + `use()` hook

#### Sibling 병렬 렌더링 (가장 큰 변화)
- 하나가 suspend되어도 **sibling 렌더링이 계속 진행**
- sibling의 data fetch가 동시에 트리거 → waterfall 해소

```tsx
// React 19: A가 suspend되어도 B 렌더링 → fetch 동시 시작
<Suspense fallback={<Loading />}>
  <ComponentA />  {/* fetch 2초 */}
  <ComponentB />  {/* fetch 3초 */}
</Suspense>
// React 18: 2+3 = 5초 (순차)
// React 19: max(2,3) = 3초 (병렬)
```

#### `use()` hook — 최초의 공식 data fetching API
- Promise나 Context를 컴포넌트 내에서 직접 읽는 공식 API
- 조건문/반복문 안에서도 호출 가능 (일반 hook 규칙의 예외)
- 전달하는 Promise는 렌더 외부에서 생성해야 함 (Server Component, route loader 등)

```tsx
import { use } from 'react';

function UserProfile({ userPromise }) {
  const user = use(userPromise);
  return <div>{user.name}</div>;
}
```

#### Server Components 안정화
- async function으로 Server Component 선언, 자동 Suspense 연동

### 버전별 비교

| 기능 | 17 | 18 | 19 |
|------|:--:|:--:|:--:|
| Code Splitting (lazy) | O | O | O |
| Concurrent Rendering | X | O | O |
| Streaming SSR | X | O | O |
| Selective Hydration | X | O | O |
| Data Fetching 공식 API | X | X (프레임워크만) | `use()` |
| Sibling 병렬 렌더링 | X | X (중단) | O (계속) |
| Server Components | X | 실험적 | 안정화 |

## 실무 적용

- React 19의 sibling 렌더링 변경으로, 기존에 순차 실행을 가정한 코드가 있다면 병렬 실행으로 바뀌면서 의도치 않은 동작이 발생할 수 있으므로 마이그레이션 시 점검 필요
- React 18에서 병렬 fetch가 필요하면 각 컴포넌트를 별도 Suspense boundary로 감싸는 workaround 사용

## 출처 / 참고

- React 공식 문서 — Suspense
- [[rsc-async-optimization]] — RSC 비동기 최적화
- [[loading-error-internals]] — Next.js에서의 Suspense 활용

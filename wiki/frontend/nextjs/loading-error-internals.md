---
title: Next.js loading/error vs Suspense/ErrorBoundary 내부 동작
aliases:
  - loading.tsx 내부 동작
  - Next.js Suspense 내부 처리
tags:
  - next-js
  - suspense
  - error-boundary
  - streaming
  - ppr
created: 2026-04-02
updated: 2026-04-02
---

## 정의

Next.js App Router의 `loading.tsx`와 `error.tsx`는 파일 컨벤션으로, 빌드 시 각각 React의 `<Suspense>`와 `<ErrorBoundary>`로 자동 변환된다. 단순한 래핑 축약이 아니라 Next.js router와 통합된 인프라 수준의 처리가 추가된다.

## 상세 설명

### 내부 컴포넌트 트리 중첩 순서

Next.js가 각 route segment마다 생성하는 트리 구조:

```
<Layout>                                    ← layout.tsx (가장 바깥)
  <Template>                                ← template.tsx
    <ErrorBoundary fallback={<Error />}>    ← error.tsx
      <Suspense fallback={<Loading />}>     ← loading.tsx
        <Page />                            ← page.tsx (가장 안쪽)
      </Suspense>
    </ErrorBoundary>
  </Template>
</Layout>
```

이 순서에서 나오는 규칙:

| 규칙 | 이유 |
|------|------|
| layout의 에러는 같은 segment의 error.tsx로 catch 불가 | layout이 ErrorBoundary 바깥 |
| layout은 loading.tsx 영향을 안 받음 | layout이 Suspense 바깥 |
| error.tsx가 loading 상태의 에러도 잡음 | ErrorBoundary가 Suspense 바깥 |

### Prefetch 참여 여부

`loading.tsx`는 Next.js **router의 prefetch 파이프라인에 참여**한다.

- `loading.tsx` 있음 → Link가 뷰포트에 진입하면 layout + loading fallback을 미리 prefetch (30초 캐시) → 클릭 시 즉시 loading UI 표시
- `loading.tsx` 없음 → prefetch 제한적 → 클릭 시 서버 응답까지 이전 페이지 유지
- 직접 `<Suspense>` → router가 인식 못함 → prefetch 대상 아님

### Route 전환 시 자동 remount

- `loading.tsx`의 Suspense boundary → segment 기반 key 자동 부여 → route 변경 시 remount → fallback 다시 표시
- 직접 `<Suspense>` → key 없으면 같은 컴포넌트로 간주 → 이전 데이터가 잠깐 보이는 문제 발생 가능 → `<Suspense key={param}>` 수동 관리 필요

### Streaming 단위

```
loading.tsx:     [Shell + Loading] ────────── [page 전체 한번에 swap]
직접 Suspense:   [Shell + A,B,C]  ─[A]─[B]─[C] (각각 독립 resolve)
```

`loading.tsx`는 page 전체를 하나의 chunk로 resolve. 직접 Suspense는 각 boundary가 독립적으로 점진 렌더링.

### PPR (Partial Prerendering)에서의 역할

PPR 활성화 시 Suspense boundary의 fallback이 빌드 타임에 static shell로 prerender된다.

- `loading.tsx`만 → page 전체가 하나의 dynamic hole
- 직접 `<Suspense>` → 세밀한 multiple holes 가능 → static shell에 더 많은 콘텐츠 포함

```tsx
// 직접 Suspense로 세밀한 PPR
export default function Page() {
  return (
    <div>
      <h1>제품 상세</h1>              {/* static shell */}
      <StaticDescription />            {/* static shell */}
      <Suspense fallback={<Skeleton />}>
        <DynamicReviews />              {/* dynamic hole */}
      </Suspense>
    </div>
  )
}
```

### error.tsx의 reset() 특수 동작

| | error.tsx | 직접 ErrorBoundary |
|---|---|---|
| `reset()` | error state만 클리어 (서버 컴포넌트 re-fetch 안 함) | 동일 |
| 서버 컴포넌트 에러 | Next.js가 직렬화하여 전달 | 접근 불가 (client-only) |
| `router.refresh()` 연동 | 같은 segment에 바인딩 | 수동 구현 필요 |

서버 에러 복구 시 `startTransition`으로 `router.refresh()` + `reset()`을 묶어야 한다:

```tsx
'use client'
export default function Error({ error, reset }) {
  const router = useRouter()
  return (
    <button onClick={() => {
      startTransition(() => {
        router.refresh()  // 서버 컴포넌트 캐시 무효화
        reset()           // error state 클리어
      })
    }}>
      재시도
    </button>
  )
}
```

### 종합 비교

| 차원 | loading.tsx / error.tsx | 직접 Suspense / ErrorBoundary |
|------|:---:|:---:|
| Prefetch 참여 | O | X |
| Instant navigation | O | X |
| Route 변경 시 자동 remount | O | X (수동 key) |
| PPR static shell | page 전체 1개 hole | 세밀한 multiple holes |
| Streaming 단위 | page 전체 | 컴포넌트별 독립 |
| 제어 범위 | segment 단위 (coarse) | 컴포넌트 단위 (fine) |

## 실무 적용

`loading.tsx`로 route 전환의 기본 UX(instant loading)를 확보하고, 페이지 내부에서는 직접 `<Suspense>`로 점진적 렌더링을 구현하는 것이 best practice. 둘은 상호 보완적이다.

## 출처 / 참고

- Next.js 공식 문서 — loading.js, error.js
- [[rendering-lifecycle|Next.js 15 렌더링 생애주기]]
- [[caching-and-data-fetching|Next.js 15 캐싱 & 데이터 페칭]]
- [[rsc-async-optimization|RSC 비동기 최적화]]
- [[suspense-version-diff|React Suspense 버전별 동작 차이]]

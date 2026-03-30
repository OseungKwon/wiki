---
title: Next.js 15 렌더링 생애주기
aliases:
  - Next.js 렌더링 파이프라인
  - RSC 렌더링 순서
tags:
  - next-js
  - react
  - rsc
  - rendering
  - streaming
  - hydration
created: 2026-03-30
updated: 2026-03-30
---

## 정의

Next.js 15의 App Router 기반 렌더링 생애주기. 서버에서 RSC가 실행되고 HTML이 스트리밍되며, 클라이언트에서 hydration 후 인터랙티브해지는 전체 흐름을 다룬다.

## 상세 설명

### 전체 요청 흐름

```
[Browser] → URL 요청
    ↓
[Edge] middleware.ts (리다이렉트/리라이트/헤더 수정)
    ↓
[Server] Route Resolution (App Router 파일 시스템)
    ↓
[Server] Layout 트리 순회 (root → 중첩 layout → page)
    ↓
[Server] Server Components 실행 (async, top-down)
         ├── fetch() 호출
         ├── DB 쿼리
         ├── generateMetadata() (페이지와 병렬 실행)
         └── Suspense 경계에서 스트리밍 청크 생성
    ↓
[Server → Browser] HTML 스트림 시작 (PPR이면 정적 셸 먼저)
    ↓
[Browser] 초기 HTML 페인트
    ↓
[Browser] JS 번들 로드 (client components)
    ↓
[Browser] React hydration (이벤트 리스너 부착)
    ↓
[Browser] useEffect 실행, 클라이언트 인터랙티브
```

### 서버사이드 실행 순서

```
Request
  → middleware.ts (Edge에서 가장 먼저 실행)
    → Route resolution
      → Root layout.tsx (서버 컴포넌트)
        → 중첩 layout.tsx (서버 컴포넌트)
          → loading.tsx (Suspense 경계 제공)
            → page.tsx (서버 컴포넌트)
              → generateMetadata() (가능하면 page와 병렬)
```

- **Server Components**는 서버에서 **top-down** 실행, React 19에서 `async` 함수 가능
- `'use client'` 경계를 만나면 **참조만** RSC payload에 직렬화 (코드는 별도 번들)
- **HTML Streaming** — `renderToReadableStream`으로 스트리밍. Suspense fallback 먼저 전송 후 완료 시 `<script>` 태그로 교체
- **Server Actions** — `'use server'` 함수는 POST 엔드포인트로 컴파일됨. 호출 시 POST → 서버 실행 → RSC payload 응답 → 클라이언트 reconcile

### 라우팅 컴포넌트 래핑 순서 (Inside Out)

```jsx
<Layout>              {/* layout.tsx — 네비게이션 간 유지, 리마운트 안 됨 */}
  <Template>          {/* template.tsx — 매 네비게이션마다 리마운트 */}
    <ErrorBoundary fallback={<Error />}>    {/* error.tsx */}
      <Suspense fallback={<Loading />}>     {/* loading.tsx */}
        <ErrorBoundary fallback={<NotFound />}>  {/* not-found.tsx */}
          <Page />                                {/* page.tsx */}
        </ErrorBoundary>
      </Suspense>
    </ErrorBoundary>
  </Template>
</Layout>
```

### Hydration 순서 (클라이언트)

1. 초기 HTML 도착 → 브라우저 즉시 페인트
2. JS 번들 로드 (React 런타임 + 클라이언트 컴포넌트)
3. `hydrateRoot()`로 이벤트 리스너 부착 (React 19: 3rd party 스크립트 mismatch 자동 억제)
4. `'use client'` 컴포넌트의 `useEffect`/`useState` 활성화
5. 미완료 Suspense 경계가 스트리밍 데이터로 해소

### 소프트 네비게이션 (클라이언트 사이드)

1. `<Link>` 뷰포트 진입 시 RSC payload 프리페치
2. 네비게이션 시 새 route의 RSC payload 페치
3. React가 **변경된 세그먼트만** reconcile — Layout 유지
4. `history.pushState()`로 URL 업데이트
5. `loading.tsx` Suspense fallback이 즉시 표시 (미완료 시)

### Partial Prerendering (PPR) — 실험적

1. **빌드 시**: 정적 셸 프리렌더 (layout, Suspense 위의 정적 콘텐츠)
2. **요청 시**: `<Suspense>` 내부 동적 콘텐츠 스트리밍
3. **단일 HTTP 요청**: 정적 셸 즉시 반환 + 동적 부분 스트리밍
4. 설정: `experimental: { ppr: 'incremental' }` + `export const experimental_ppr = true`

### React 19 통합 영향

- `async` Server Components 일급 지원
- `ref`가 일반 prop — `forwardRef` 불필요
- `use()` 훅 — 렌더 중 Promise/Context unwrap
- `useActionState` (`useFormState` 대체) — form 제출 상태 관리
- `useOptimistic` — Server Action과 함께 낙관적 UI 업데이트
- Document Metadata — 컴포넌트 내 `<title>`, `<meta>` 자동 `<head>` 호이스팅

## 출처 / 참고

- [Next.js Rendering 공식 문서](https://nextjs.org/docs/app/building-your-application/rendering)
- [Next.js 15 블로그](https://nextjs.org/blog/next-15)
- [[caching-and-data-fetching|Next.js 15 캐싱 & 데이터 페칭]]
- [[infrastructure-and-deployment|Next.js 15 인프라 & 배포]]
- [[rsc-async-optimization|RSC 비동기 최적화]]

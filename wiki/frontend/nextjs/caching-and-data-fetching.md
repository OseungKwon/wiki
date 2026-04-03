---
title: Next.js 15 캐싱 & 데이터 페칭
aliases:
  - Next.js 캐시 전략
  - Next.js fetch 캐시
tags:
  - next-js
  - caching
  - data-fetching
  - isr
  - revalidation
created: 2026-03-30
updated: 2026-03-30
reviewed: false
---

## 정의

Next.js 15의 4가지 캐시 레이어와 데이터 페칭 동작 방식. Next.js 14 대비 캐시 기본값이 대폭 변경되어 **기본적으로 캐시하지 않는** 방향으로 전환되었다.

## 상세 설명

### Next.js 15 캐시 기본값 변경 (vs 14)

| 동작 | Next.js 14 | Next.js 15 |
|------|------------|------------|
| `fetch()` 기본값 | `force-cache` (캐시됨) | **`no-store` (캐시 안 됨)** |
| `GET` Route Handler | 캐시 기본 | **캐시 안 됨** |
| Client Router Cache (`staleTime`) | 30s dynamic / 5min static | **0s dynamic** / 5min static |

### 4가지 캐시 레이어

#### 1. Full Route Cache
- **대상**: 정적 route의 프리렌더된 HTML + RSC Payload
- **저장소**: `.next/server/app/` (파일시스템)
- **무효화**: `revalidatePath()`, `revalidateTag()`, 재배포
- **Next.js 15**: `cookies()`, `headers()`, `searchParams` 사용 route는 항상 dynamic

#### 2. Data Cache (서버 fetch 캐시)
- **대상**: Server Component/Route Handler의 `fetch()` 응답
- **저장소**: `.next/cache/fetch-cache/` (파일시스템)
- **Next.js 15 기본**: 캐시 안 됨. 명시적 opt-in 필요:

```typescript
// 영구 캐시
fetch(url, { cache: 'force-cache' })

// 시간 기반 재검증
fetch(url, { next: { revalidate: 3600 } })

// 태그 기반 (온디맨드 무효화용)
fetch(url, { next: { tags: ['products'] } })
```

- **커스텀 캐시 핸들러**: Redis 등으로 교체 가능 (`cacheHandler` 설정)

#### 3. Router Cache (클라이언트)
- **대상**: 방문한 route의 RSC payload (브라우저 메모리)
- **Next.js 15**: page segment `staleTime` = **0** (매번 리페치), layout은 30s 유지
- **설정 변경**:

```js
// next.config.js
module.exports = {
  experimental: {
    staleTimes: {
      dynamic: 0,    // Next.js 15 기본
      static: 300,   // prefetch된 route 5분
    }
  }
}
```

#### 4. Image Cache
- **저장소**: `.next/cache/images/`
- 요청 시 온디맨드 최적화 (빌드 시 아님), Accept 헤더 기반 WebP/AVIF 서빙

### Request Deduplication

동일 렌더 패스 내 같은 URL의 `fetch()` 호출은 자동 중복 제거. `layout.tsx`와 `page.tsx`가 같은 URL을 fetch해도 1회만 실행.

### Revalidation 동작

- **시간 기반**: `{ next: { revalidate: 60 } }` — stale-while-revalidate 패턴
- **온디맨드**: Server Action/Route Handler에서 `revalidatePath()` / `revalidateTag()`
- **흐름**: stale 응답 즉시 반환 → 백그라운드 재생성 → 다음 요청부터 새 콘텐츠

### `params`/`searchParams` async 변경

```typescript
// Next.js 14
function Page({ params }: { params: { id: string } }) { ... }

// Next.js 15 — Promise로 변경 (PPR 최적화)
async function Page({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params
}
```

params가 resolve되기 전에 페이지 셸 렌더링을 시작할 수 있어 PPR에 핵심적.

### 실험적: `'use cache'` 디렉티브

```typescript
'use cache'

export async function getProducts() {
  // cacheLife, cacheTag으로 세밀한 캐시 제어
  return db.query('SELECT * FROM products')
}
```

fetch 레벨 캐시 옵션의 대안으로 함수 단위 캐시 제어 가능.

## 출처 / 참고

- [Next.js Caching 공식 문서](https://nextjs.org/docs/app/building-your-application/caching)
- [Next.js Data Fetching 공식 문서](https://nextjs.org/docs/app/building-your-application/data-fetching)
- [[rendering-lifecycle|Next.js 15 렌더링 생애주기]]
- [[infrastructure-and-deployment|Next.js 15 인프라 & 배포]]

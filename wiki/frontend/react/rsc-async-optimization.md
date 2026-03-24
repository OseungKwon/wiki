---
title: RSC 비동기 최적화
aliases:
  - React Server Components async optimization
  - RSC data fetching
  - 서버 컴포넌트 비동기 최적화
tags:
  - react
  - rsc
  - server-components
  - async
  - performance
  - next-js
  - caching
created: 2026-03-24
updated: 2026-03-24
---

## 정의
React Server Components(RSC)에서 비동기 데이터 페칭의 waterfall을 제거하고, streaming·캐싱·중복 제거를 통해 성능을 극대화하는 패턴들의 모음.

## 핵심 원리: 서버에서 fetch하면 왜 빠른가

RSC는 데이터를 서버에서 직접 가져오므로 클라이언트 ↔ API 간 네트워크 왕복이 제거된다.

| 방식 | LCP | 데이터 완료 |
|------|-----|------------|
| Client-Side fetch | 4.1s | 5.1s |
| SSR + client fetch | 1.28s | 4.9s |
| **RSC (server fetch)** | **1.28s** | **1.28s** |

> 성능 개선은 **데이터 페칭이 관여할 때만** 의미가 있다. 정적 앱은 기존 SSR과 차이 없음.

## 상세 설명

### 1. 병렬 페칭 (Parallel Fetching)

RSC 내에서 순차 `await`는 여전히 waterfall을 만든다.

```tsx
// ❌ 순차 — getAlbums는 getArtist 완료까지 대기
const artist = await getArtist(username);
const albums = await getAlbums(username);

// ✅ 병렬 — 동시에 시작, 동시에 await
const artistData = getArtist(username);
const albumsData = getAlbums(username);
const [artist, albums] = await Promise.all([artistData, albumsData]);
```

**레이아웃 vs 페이지**: Next.js는 layout과 page를 자동으로 병렬 렌더링한다. 단, 같은 컴포넌트 내 여러 `await`는 수동으로 병렬화해야 한다.

### 2. React `cache()` — 요청 단위 중복 제거

```tsx
import { cache } from 'react';

export const getUser = cache(async (id: string) => {
  return await db.user.findUnique({ where: { id } });
});
```

**동작 원리:**
- 같은 서버 요청(request) 내에서 동일 인자 호출 시 **1번만 실행**
- `Object.is`로 인자를 비교 (shallow equality)
- Promise 자체가 캐시됨 → 비동기 작업도 중복 제거
- **요청이 끝나면 캐시 소멸** (cross-request 캐시 아님)

**Preload 패턴과 조합:**
```tsx
const getItem = cache(async (id: string) => {
  return await fetch(`/api/items/${id}`);
});

// 상위 컴포넌트: await 없이 Promise만 시작
function Page({ id }) {
  getItem(id); // fire-and-forget → 캐시에 Promise 등록
  return <ItemDetail id={id} />;
}

// 하위 컴포넌트: 캐시 히트
async function ItemDetail({ id }) {
  const item = await getItem(id); // 이미 진행 중이거나 완료된 Promise
  return <div>{item.name}</div>;
}
```

**주의사항:**
- **서버 컴포넌트 전용** (`'use client'`에서 사용 불가)
- 모듈 레벨에서 한 번만 `cache()` 호출해야 함 (컴포넌트 내부에서 매번 호출하면 별개 캐시)
- 객체 인자는 **참조 동일성** 필요 → 가능하면 primitive 전달

### 3. Streaming + Suspense — 점진적 렌더링

준비된 콘텐츠를 먼저 보내고, 느린 데이터는 나중에 스트리밍:

```tsx
// 즉시 표시되는 부분
export default function Page() {
  return (
    <div>
      <h1>대시보드</h1>
      <Suspense fallback={<PostsSkeleton />}>
        <SlowPosts />  {/* 이 부분만 나중에 스트리밍 */}
      </Suspense>
    </div>
  );
}

async function SlowPosts() {
  const posts = await getSlowData(); // 이 컴포넌트만 기다림
  return <PostList posts={posts} />;
}
```

**`loading.js`**: 페이지 단위 스트리밍 (자동 Suspense 래핑)
**`<Suspense>`**: 컴포넌트 단위 세밀한 제어 (권장)

**Server → Client 스트리밍 (`use` API):**
```tsx
// Server Component: await 없이 Promise 전달
export default function Page() {
  const posts = getPosts(); // Promise
  return (
    <Suspense fallback={<Spinner />}>
      <Posts posts={posts} />
    </Suspense>
  );
}

// Client Component: use()로 resolve
'use client'
import { use } from 'react';

function Posts({ posts }: { posts: Promise<Post[]> }) {
  const allPosts = use(posts); // Suspense가 fallback 처리
  return <ul>{allPosts.map(p => <li key={p.id}>{p.title}</li>)}</ul>;
}
```

### 4. `use cache` 디렉티브 (Next.js 16+)

`cache()`가 요청 단위 메모이제이션이라면, `'use cache'`는 **cross-request 영속 캐시**.

```tsx
// 파일 레벨 — 모든 export가 캐시됨
'use cache'

export default async function Page() {
  const data = await fetch('/api/data');
  return <div>{data}</div>;
}

// 함수 레벨
export async function getData() {
  'use cache'
  return await fetch('/api/data');
}

// 컴포넌트 레벨
export async function Bookings({ type }: { type: string }) {
  'use cache'
  const data = await fetch(`/api/bookings?type=${type}`);
  return <div>{data}</div>;
}
```

**캐시 키 자동 생성:** Build ID + 함수 ID + 직렬화된 인자 + 클로저 변수

**캐시 수명 제어:**
```tsx
import { cacheLife, cacheTag } from 'next/cache';

async function getProducts() {
  'use cache'
  cacheLife('hours')       // 내장 프로필: hours, days 등
  cacheTag('products')     // on-demand 무효화용 태그
  return await fetch('/api/products');
}
```

**On-demand 무효화:**
```tsx
'use server'
import { revalidateTag } from 'next/cache';

export async function updateProduct() {
  await db.products.update(...);
  revalidateTag('products'); // 'products' 태그된 모든 캐시 무효화
}
```

**Interleaving — 캐시/동적 컴포넌트 조합:**
```tsx
export default async function Page() {
  const uncachedData = await getData();
  return (
    <CachedShell>
      <DynamicComponent data={uncachedData} /> {/* pass-through: 캐시 영향 X */}
    </CachedShell>
  );
}

async function CachedShell({ children }) {
  'use cache'
  const cached = await fetch('/api/static');
  return <div><StaticPart data={cached} />{children}</div>;
}
```

**PPR (Partial Prerendering):** `use cache`와 결합하여 정적 셸은 즉시 전송, 동적 부분만 스트리밍.

**제약사항:**
- `cookies()`, `headers()` 등 런타임 API 직접 접근 불가 → 외부에서 읽어 인자로 전달
- 인자와 반환값은 직렬화 가능해야 함 (클래스 인스턴스, 함수 불가)
- `React.cache`와 격리됨: `use cache` 내부에서 외부 `React.cache` 값 접근 불가

### 5. Server/Client 간 데이터 공유 패턴

`React.cache` + Context로 서버와 클라이언트 모두에서 같은 데이터를 중복 없이 사용:

```tsx
// lib/user.ts — 캐시된 fetcher
export const getUser = cache(async () => {
  return await fetch('/api/user').then(r => r.json());
});

// layout.tsx — Promise를 Context에 제공 (await 하지 않음)
export default function Layout({ children }) {
  const userPromise = getUser();
  return <UserProvider userPromise={userPromise}>{children}</UserProvider>;
}

// Server Component — 직접 await (캐시 히트)
const user = await getUser();

// Client Component — use()로 resolve (같은 Promise)
const userPromise = useContext(UserContext);
const user = use(userPromise);
```

## 전략 비교

| 전략 | 범위 | 수명 | 용도 |
|------|------|------|------|
| `Promise.all` | 컴포넌트 | 없음 | 독립 요청 병렬화 |
| `React.cache()` | 단일 요청 | 요청 종료 시 소멸 | 중복 제거 + preload |
| `<Suspense>` + Streaming | 컴포넌트 트리 | 없음 | 점진적 렌더링 |
| `'use cache'` | cross-request | 설정 가능 (hours, days 등) | 영속 캐시 + PPR |
| `use()` API | Client Component | 없음 | 서버→클라이언트 Promise 전달 |

## 출처 / 참고
- [React cache() 공식 문서](https://react.dev/reference/react/cache)
- [Next.js Data Fetching](https://nextjs.org/docs/app/getting-started/fetching-data)
- [Next.js `use cache` 디렉티브](https://nextjs.org/docs/app/api-reference/directives/use-cache)
- [RSC Streaming Performance Guide](https://www.sitepoint.com/react-server-components-streaming-performance-2026/)
- [Intro to Performance of RSC](https://calendar.perfplanet.com/2025/intro-to-performance-of-react-server-components/)

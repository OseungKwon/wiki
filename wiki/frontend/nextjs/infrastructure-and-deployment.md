---
title: Next.js 15 인프라 & 배포
aliases:
  - Next.js 배포 모델
  - Next.js 빌드 출력
tags:
  - next-js
  - infrastructure
  - deployment
  - docker
  - edge
  - streaming
  - monitoring
created: 2026-03-30
updated: 2026-03-30
reviewed: false
---

## 정의

Next.js 15의 인프라 관점 생애주기. 서버 시작, 런타임 선택, 빌드 출력 구조, 배포 모델, 스트리밍 인프라, 모니터링까지 포함한다.

## 상세 설명

### 서버 시작 순서

```
프로세스 시작
  → instrumentation.ts 로드
  → register() 호출 (1회 — DB pool, APM, Sentry 초기화)
  → Next.js HTTP 서버 초기화
  → 요청 수락 준비 완료
```

```typescript
// instrumentation.ts (Next.js 15에서 stable)
export async function register() {
  if (process.env.NEXT_RUNTIME === 'nodejs') {
    await import('./sentry.server.config')
  }
}

export async function onRequestError(error, request, context) {
  // 모든 서버 에러 중앙 핸들링
  await Sentry.captureException(error, { extra: context })
}
```

- **self-hosted**: 단일 Node.js 프로세스. 멀티코어는 PM2 cluster 또는 컨테이너 복제
- **serverless**: 콜드 스타트마다 `register()` 재실행

### Edge vs Node.js 런타임

| 항목 | Node.js | Edge |
|------|---------|------|
| API | 전체 Node.js API | Web API 서브셋 (`fs`, `net` 없음) |
| 콜드 스타트 | ~250-500ms | ~1-5ms |
| 최대 실행시간 | 무제한 (self-hosted) | 30s (Vercel) |
| 메모리 | 설정 가능 | 128MB (Vercel) |
| DB 연결 | TCP 직접 연결 | HTTP 기반만 |
| 배포 | 단일 리전 | 글로벌 분산 |

- `middleware.ts`는 **항상 Edge** 실행
- route별 `export const runtime = 'edge'`로 개별 설정
- Edge는 V8 isolate 기반 (Cloudflare Workers와 유사)

### 빌드 출력 구조 (`.next/`)

```
.next/
├── cache/
│   ├── fetch-cache/          # Data Cache (fetch 응답)
│   ├── images/               # 최적화된 이미지
│   └── webpack/              # 빌드 캐시 (리빌드 가속)
├── server/
│   ├── app/                  # 서버 번들 (RSC, HTML) — 민감 코드 포함 가능
│   ├── chunks/               # 서버 코드 청크
│   ├── middleware.js          # 컴파일된 미들웨어
│   └── server-reference-manifest.json  # Server Actions 매핑
├── static/
│   ├── chunks/               # 클라이언트 JS (route별 코드 스플릿)
│   ├── css/                  # 추출된 CSS
│   ├── media/                # 폰트 등 정적 에셋
│   └── <buildId>/            # 빌드별 에셋
├── BUILD_ID                  # 빌드 고유 해시
└── routes-manifest.json      # 전체 라우트 설정
```

- `_next/static/*` 파일명에 **content hash** 포함 → `Cache-Control: public, max-age=31536000, immutable` 권장
- **서버 번들**: DB 쿼리, 시크릿 포함 가능 (브라우저 전송 안 됨)
- **클라이언트 번들**: `'use client'` 컴포넌트 + React 런타임

### 코드 스플릿 전략

1. **Route 기반**: 각 route에 독립 클라이언트 JS 번들
2. **Component 기반**: `React.lazy()`, `next/dynamic`으로 추가 분할
3. **Shared 청크**: React, 공유 컴포넌트 등 중복 제거
4. 로딩: `<link rel="prefetch">` (저우선) 또는 `<script async>` (고우선)

### 배포 모델 비교

#### Self-hosted (`next start`)
- Node.js HTTP 서버 실행, 모든 기능 지원
- Nginx 리버스 프록시 + PM2 권장
- 캐시 영속성(`.next/cache/`) 직접 관리

#### Standalone (Docker 최적)

```js
module.exports = { output: 'standalone' }
```

- `@vercel/nft`로 필요 의존성만 트레이싱 → `.next/standalone/` (~50MB)
- `public/`과 `.next/static/`은 수동 복사 필요

```dockerfile
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public
CMD ["node", "server.js"]
```

#### Vercel
- 정적 에셋 → 글로벌 CDN
- 동적 route → Serverless Function
- Edge route → V8 Isolate (글로벌)
- 미들웨어 → 항상 Edge (가장 가까운 PoP)
- ISR → 분산 KV 스토어로 자동 처리

#### Static Export (`output: 'export'`)
- 순수 정적 HTML → S3, GitHub Pages 등
- ISR, Server Components, 미들웨어, 이미지 최적화 **불가**

### 스트리밍 & 로드밸런서

**HTTP 스트리밍 동작**:
1. `Transfer-Encoding: chunked`로 응답
2. Suspense fallback HTML 먼저 전송
3. 비동기 완료 시 `<script>` 태그로 교체 청크 스트림
4. **응답 시작 후 status code 변경 불가**

**로드밸런서 필수 설정**:

```nginx
# Nginx — 버퍼링 비활성화 필수
proxy_buffering off;
proxy_cache off;
proxy_http_version 1.1;
proxy_set_header Connection '';
```

- 프록시 버퍼링 시 스트리밍이 깨짐
- HTTP/2 권장, keep-alive 타임아웃 > Next.js 서버 타임아웃

### 모니터링 & 옵저버빌리티

**`next/after`** — 응답 후 백그라운드 작업 (Next.js 15 stable):

```typescript
import { after } from 'next/server'

export async function POST(request: Request) {
  const data = await processRequest(request)
  after(async () => {
    await analytics.track('api_call')  // 응답 전송 완료 후 실행
  })
  return Response.json(data)
}
```

**내장 OpenTelemetry 스팬**: `next.route`, `next.render`, `next.middleware`, `next.fetch`, `next.generateMetadata`

**에러 리포팅 흐름**:
- 서버 에러 → `onRequestError` (instrumentation.ts)
- 렌더 에러 → `error.tsx` 경계
- 루트 레이아웃 에러 → `global-error.tsx`
- `after()` 내 에러 → 로그만 남고 응답에 영향 없음

### 커넥션 풀링 (자체 관리 필요)

```typescript
// lib/db.ts — global singleton 패턴
const globalForDb = globalThis as unknown as { pool: Pool }
export const pool = globalForDb.pool ?? new Pool({ max: 20 })
if (process.env.NODE_ENV !== 'production') globalForDb.pool = pool
```

- Serverless 환경: PgBouncer, Neon serverless driver 등 커넥션 풀러 필수
- `instrumentation.ts`의 `register()`에서 초기화 권장

## 출처 / 참고

- [Next.js Deploying 공식 문서](https://nextjs.org/docs/app/building-your-application/deploying)
- [Next.js 15 블로그](https://nextjs.org/blog/next-15)
- [[rendering-lifecycle|Next.js 15 렌더링 생애주기]]
- [[caching-and-data-fetching|Next.js 15 캐싱 & 데이터 페칭]]
- [[edge-computing|프론트엔드 Edge Computing]]

---
title: 프론트엔드 Edge Computing
aliases:
  - Edge Functions
  - Edge Middleware
  - Edge Runtime
tags:
  - edge
  - middleware
  - serverless
  - deployment
  - performance
created: 2026-03-24
updated: 2026-03-24
---

## 정의

Edge Computing은 사용자와 가장 가까운 CDN 엣지 서버에서 코드를 실행하는 방식이다. "Edge"는 네트워크의 가장자리(edge)를 의미하며, Origin 서버(중앙)가 아닌 전 세계에 분산된 노드에서 실행되어 지연 시간을 최소화한다.

## 상세 설명

### 요청 흐름과 계층 구조

```
사용자 요청
  ↓
[Edge Middleware]       ← 라우팅 전 실행 (리다이렉트, 인증, 헤더)
  ↓
[Edge Functions]        ← 범용 엣지 실행 (API, SSR, 개인화)
  ↓
[Serverless Functions]  ← Origin 리전 (DB 쿼리, 무거운 로직)
  ↓
[Origin Server]         ← 전통적 서버
```

### Edge vs Serverless vs Origin 비교

| 구분 | Edge Middleware | Edge Functions | Serverless |
|---|---|---|---|
| 위치 | CDN 엣지 | CDN 엣지 | 특정 리전 |
| 콜드스타트 | ~0ms | <5ms (Cloudflare) | 100ms~1s+ |
| 런타임 | Web API subset | Web API subset | 풀 Node.js |
| 용도 | 라우팅/리다이렉트 | API, SSR, 개인화 | DB, 복잡한 연산 |
| 제약 | 가장 제한적 | Node 네이티브 API 불가 | 거의 없음 |

### Edge Runtime 제약사항

Edge는 **V8 Isolate** 기반으로 Node.js가 아니다:

```
✅ fetch, Request, Response, crypto, TextEncoder, URL, Headers
❌ fs, path, child_process, net, Buffer(일부), node-gyp 패키지
```

`node_modules` 중 Node.js 네이티브 API에 의존하는 패키지는 Edge에서 동작하지 않는다.

### 프레임워크별 현황

**Next.js**
- `middleware.ts` (루트): 모든 요청 전에 Edge에서 실행
- Route Handler/Page에 `export const runtime = 'edge'` 설정으로 Edge Runtime 선택
- Vercel은 2025년 중반부터 "Edge Functions" → "Vercel Functions (Edge Runtime)"으로 통합

**React Router v7 (Remix)**
- v7.9.0에서 Middleware가 stable로 추가 (`future.v8_middleware` 플래그)
- 단, 서버 미들웨어이며 Edge 특화는 아님. Edge 배포는 어댑터 의존 (Cloudflare Workers 등)
- 미들웨어는 해당 라우트 + 모든 하위 라우트에 영향

**Cloudflare Workers**
- 콜드스타트 5ms 미만, Lambda@Edge 대비 약 9배 빠름
- Workers AI (2025년 말): 엣지에서 AI 추론 실행
- WebAssembly 지원으로 Rust, Go, C++ 코드도 엣지 실행 가능

### Edge에 적합한 작업

```
✅ JWT 토큰 검증
✅ 지역 기반 리다이렉트 (geo-routing)
✅ A/B 테스트 분기
✅ Bot 감지 / Rate limiting
✅ 헤더 조작 (CORS, CSP)
✅ 쿠키 기반 개인화
```

### Origin/Serverless에 맡겨야 할 작업

```
❌ DB 쿼리 (커넥션 풀 필요)
❌ 파일 시스템 접근
❌ 무거운 연산 (이미지 처리 등)
❌ Node.js 네이티브 의존 라이브러리
```

### 2026년 트렌드

- **Wasm at the Edge**: JS 외에 Rust/Go로 엣지 로직 작성 증가
- **Edge + AI**: Cloudflare Workers AI처럼 추론을 엣지에서 실행
- **통합 런타임**: Vercel처럼 Edge/Serverless 구분을 플랫폼이 자동 결정하는 방향

## 실무 적용

현재 프로젝트(passorder-web)는 React Router v7 기반이므로 전용 Edge Middleware가 없다. 인증/리다이렉트 등의 공통 로직은 상위 레이아웃 route의 `loader`/`clientLoader`에서 처리하고 있다. Edge 배포가 필요하면 Cloudflare Workers 어댑터를 도입하거나, v7.9.0+의 `future.v8_middleware`를 활용할 수 있다.

## 출처 / 참고

- [Edge Computing Frontend 2026 Guide](https://www.live-laugh-love.world/blog/edge-computing-frontend-developers-guide-2026/)
- [Edge Functions vs Serverless in Next.js 2025](https://medium.com/@itsamanyadav/edge-functions-vs-serverless-in-next-js-which-one-should-you-use-in-2025-f4ae28c0788d)
- [Remix Blog - Middleware in React Router](https://remix.run/blog/middleware)
- [Edge Runtime vs Node.js Runtime](https://dev.to/pockit_tools/edge-runtime-vs-nodejs-runtime-when-your-serverless-functions-mysteriously-fail-14a)
- [[loader-action|React Router v7 - Loader & Action]]

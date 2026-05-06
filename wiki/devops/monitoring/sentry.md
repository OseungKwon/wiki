---
title: "Sentry - 프론트엔드 에러 모니터링"
aliases:
  - Sentry
  - 센트리
tags:
  - sentry
  - monitoring
  - error-tracking
  - javascript
  - observability
created: 2026-05-06
updated: 2026-05-06
reviewed: false
---

## 정의

Sentry는 애플리케이션의 에러를 실시간으로 추적·분석하는 오픈소스 모니터링 플랫폼이다. 에러 발생 시 Stack Trace, Breadcrumbs, Context를 자동 수집하여 재현 없이도 원인을 추적할 수 있게 해준다.

## 상세 설명

### 에러 수집 원리 — Monkey Patching

Sentry SDK의 핵심은 **브라우저 네이티브 API를 감싸서(wrap) 가로채는 것**이다. `Sentry.init()` 호출 시 다음 API들을 monkey patch한다:

| 대상 API | 목적 |
|----------|------|
| `window.onerror` | 일반 JS 에러 캡처 |
| `window.onunhandledrejection` | Promise rejection 캡처 |
| `fetch` / `XMLHttpRequest` | HTTP 요청/응답 기록 (Breadcrumbs + HTTP 에러) |
| `EventTarget.addEventListener` | DOM 클릭/키입력 기록 (Breadcrumbs) |
| `console.*` | 콘솔 로그 기록 (Breadcrumbs) |
| `history.pushState` | SPA 네비게이션 기록 (Breadcrumbs) |

원래 함수를 감싸서, **원래 동작은 그대로 수행하되 정보만 추가로 기록**하는 방식이다.

### 수집하는 3가지 데이터

#### 1. Stack Trace
에러 발생 시 `error.stack`에서 호출 스택을 파싱한다. 프로덕션에서는 코드가 minify되어 있으므로, **Source Map**을 Sentry 서버에 업로드하면 서버 측에서 원본 파일명·줄번호·함수명으로 역매핑한다.

```
minified:  at a (app.min.js:1:4021)
           ↓ source map 매핑
original:  at processOrder (src/order/process.ts:12:5)
```

#### 2. Breadcrumbs
에러 발생 **직전** 사용자의 행동을 타임라인으로 기록한다:

```
[14:30:01] UI Click → "주문하기 버튼"
[14:30:02] HTTP Request → POST /api/orders (200 OK)
[14:30:03] Navigation → /orders/confirm
[14:30:04] HTTP Request → GET /api/payment (500 Error)
[14:30:04] ❌ Error → "TypeError: Cannot read property 'id' of undefined"
```

fetch/XHR, addEventListener, console, history.pushState 등을 감싸서 자동 기록한다.

#### 3. Context
브라우저 API를 직접 읽어서 수집한다 (monkey patching 아님):
- `navigator.userAgent` → 브라우저명, OS 정보
- `screen.width/height` → 화면 해상도
- `window.location.href` → 현재 URL
- 사용자 정보 (설정 시)

### 이벤트 전송 흐름

```
에러 발생
  → SDK가 가로챔 (window.onerror)
  → Stack Trace + Breadcrumbs + Context 수집
  → Envelope 형태로 묶음
  → Sentry 서버로 HTTP 전송
  → 서버에서 Source Map으로 역매핑
  → Fingerprint 생성 → 같은 이슈로 그룹화
```

### Fingerprinting (이슈 그룹화)

같은 에러가 1000번 발생해도 **1개 이슈 + 1000 events**로 관리된다. Stack Trace, Exception 타입, 메시지를 기반으로 fingerprint를 생성하고, 동일한 fingerprint는 하나의 이슈로 묶인다. 커스텀 fingerprint 설정도 가능하다.

### 성능 모니터링

에러 외에도 성능 데이터를 수집한다:
- 페이지 로드 시간, API 요청 지연시간
- Web Vitals (LCP, FID, CLS)
- 커스텀 트랜잭션/스팬

### OpenTelemetry와의 관계

Sentry는 내부적으로 OTel 프로토콜을 수용하는 방향으로 발전 중이다. 에러 추적 + 성능 모니터링이 목적이라면 Sentry가 프론트엔드에서 더 실용적이고, OTel은 백엔드 분산 추적이 주 강점이다.

| | Sentry | OpenTelemetry |
|---|--------|---------------|
| 에러 추적 | 핵심 기능 | 가능하지만 별도 설정 필요 |
| 프론트엔드 성숙도 | 높음 | experimental |
| 설정 난이도 | 쉬움 | 상대적으로 복잡 |
| 벤더 종속 | Sentry에 종속 | 벤더 중립 |

## 출처 / 참고

- [Sentry 공식 문서](https://docs.sentry.io/)
- [window.onerror로 JS 에러 캡처 - Sentry Blog](https://blog.sentry.io/client-javascript-reporting-window-onerror/)
- [Sentry Breadcrumbs 공식 문서](https://docs.sentry.io/product/issues/issue-details/breadcrumbs/)
- [Source Maps - Sentry 공식 문서](https://docs.sentry.io/platforms/javascript/sourcemaps/)
- [Sentry로 우아하게 프론트엔드 에러 추적하기 - 카카오페이](https://tech.kakaopay.com/post/frontend-sentry-monitoring/)

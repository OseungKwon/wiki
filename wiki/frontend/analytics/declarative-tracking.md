---
title: Declarative Tracking - 선언적 트래킹 패턴
aliases:
  - data-track
  - 선언적 트래킹
tags:
  - analytics
  - tracking
  - dom
  - mutation-observer
  - intersection-observer
  - design-pattern
created: 2026-03-23
updated: 2026-03-23
reviewed: false
---

## 정의

`data-track` DOM 속성 기반으로 트래킹을 선언적으로 처리하는 패턴. 컴포넌트에서 트래킹 함수를 직접 호출하지 않고, HTML 속성만으로 이벤트를 수집한다.

## 상세 설명

### 핵심 아이디어

컴포넌트 코드와 트래킹 로직을 분리한다. 트래킹 대상 요소에 `data-track="key"` 속성만 붙이면, 글로벌 옵저버가 자동으로 감지하여 등록된 핸들러를 실행한다.

```tsx
<!-- 컴포넌트에 트래킹 코드 없음 -->
<button data-track="feature.touch_cta">구매하기</button>
<section data-track="feature.content_view_banner">배너 영역</section>
```

### 트리거 유형

| 트리거 | 감지 방식 | 발화 조건 |
|---|---|---|
| `click` | document 캡처 리스너 + `closest()` | 매 클릭 |
| `pageview` | MutationObserver | DOM에 연결된 직후 1회 |
| `content-view` | IntersectionObserver (threshold 50%) | 뷰포트 노출 1회 |
| `scroll-depth` | scroll/resize 이벤트 | 마일스톤 도달 시 1회 |

### 아키텍처 구성

```
TrackingMap (선언)  →  Observer (감지)  →  Dispatcher (분배)  →  Adapter (실행)
  key → entry           MutationObserver      adapter.execute()     GA, 자체 API 등
  trigger, targets       IntersectionObserver
                         click delegation
```

**TrackingMap**: feature별로 key-entry 쌍을 정의. `satisfies TrackingMap`으로 타입 안전성 확보.

```ts
export const myTrackingMap = {
  'feature.touch_cta': {
    trigger: 'click',
    targets: [
      { type: 'user-action-history', recordType: ..., className: '...', methodName: '...' },
      { type: 'ga', eventName: 'touch_cta', params: { event_category: 'feature' } },
    ],
  },
} satisfies TrackingMap;
```

**Observer**: `initDeclarativeTracking()`이 MutationObserver로 `[data-track]` 요소 추가를 감시. 트리거 유형에 따라 즉시 발화(pageview), IO 등록(content-view), 스크롤 리스너 등록(scroll-depth).

**Dispatcher**: adapter 레지스트리에서 `target.type`으로 어댑터를 찾아 실행. 새 트래킹 플랫폼 추가 시 어댑터만 등록하면 됨.

### 동적 파라미터

```tsx
<div data-track="feature.pageview" data-track-params='{"referral":"kakao"}' />
```

JSON 문자열로 전달. GA params / 자체 API metadata에 merge된다.

### 명령형 fallback

DOM 없이도 사용 가능한 `track()` 함수 제공:

```ts
track('feature.touch_cta');
track('feature.pageview', { ga: { params: { param_01: 'value' } } }); // 타깃별 override
```

### 업계 유사 사례

| 프로젝트 | 방식 | 차이점 |
|---|---|---|
| [Google autotrack eventTracker](https://github.com/googleanalytics/autotrack) | `ga-on="click"` + `ga-*` 속성 | click만 지원, GA 전용 |
| [Trys Mudford의 Declarative Tracking](https://www.trysmudford.com/blog/declarative-tracking/) | `data-track` + document 위임 | click/submit만, 단일 provider |
| [NYT react-tracking](https://github.com/nytimes/react-tracking) | `@track()` 데코레이터 / `useTracking()` | HOC/훅 기반, DOM 속성 아님 |
| [NFL react-metrics](https://github.com/nfl/react-metrics) | `data-metrics-*` 속성 | 유사하나 IO/MO 미사용 |

### 장점

- **관심사 분리**: 컴포넌트에 트래킹 코드가 침투하지 않음
- **선언적**: HTML만 보면 어떤 트래킹이 걸려있는지 파악 가능
- **자동 감지**: MutationObserver로 동적 렌더링된 요소도 자동 처리
- **다중 타깃**: 하나의 key로 GA + 자체 API 동시 발송
- **뷰포트 트래킹 통합**: content-view, scroll-depth까지 단일 API로 처리

### 주의점

- SSR 환경에서는 클라이언트 전용으로 초기화해야 함 (`useEffect` 또는 `'use client'`)
- `data-track-params`의 JSON 직렬화 비용 (대량 리스트 렌더링 시)
- MutationObserver 콜백은 rAF로 배치 처리하여 성능 보호 필요

## 출처 / 참고

- [Google autotrack - eventTracker](https://github.com/googleanalytics/autotrack/blob/master/docs/plugins/event-tracker.md)
- [Trys Mudford - Declarative Tracking](https://www.trysmudford.com/blog/declarative-tracking/)
- [NYT react-tracking](https://github.com/nytimes/react-tracking)
- [LogRocket - react-tracking 소개](https://blog.logrocket.com/react-tracking-declarative-tracking-react-apps/)

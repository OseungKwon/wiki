---
title: "프론트엔드 트래킹, 선언적으로 풀어보니"
aliases: [선언적 트래킹 블로그, tracking blog]
tags:
  - writing
  - blog
  - analytics
  - tracking
  - dom
created: 2026-04-24
updated: 2026-04-24
reviewed: false
---

# 프론트엔드 트래킹, 선언적으로 풀어보니

프론트엔드에서 트래킹 코드가 비즈니스 로직을 잡아먹는 순간이 온다. 처음엔 클릭 한두 개 보내는 거라 대수롭지 않은데, GA 이벤트가 30개 넘고 자체 로그 서버까지 붙으면 컴포넌트마다 `onClick` 안에 트래킹 코드가 덕지덕지 붙어 있다. 리뷰할 때 "이거 트래킹 빼면 세 줄인데요"라는 말이 나오기 시작하면, 뭔가 잘못됐다는 신호다.

이 문제를 `@common/tracking`이라는 사내 패키지로 풀어봤다.

## 트래킹 코드가 컴포넌트를 망가뜨리는 과정

보통 이렇게 시작한다.

```tsx
function BannerButton() {
  const handleClick = () => {
    gtag('event', 'banner_click', { campaign: 'summer' });
    amplitude.track('Banner Click', { campaign: 'summer' });
    // 실제 로직...
  };
  return <button onClick={handleClick}>배너</button>;
}
```

간단해 보이지만 문제가 금방 커진다. GA 말고 Amplitude도 보내야 하고, 페이지뷰 이벤트도 필요하고, 스크롤 깊이도 측정해야 한다. 컴포넌트에 `useEffect`가 추가되고, `IntersectionObserver`를 직접 쓰는 훅이 생기고, 어느 순간 트래킹이 컴포넌트의 주인이 된다.

채널이 바뀔 때가 더 귀찮다. GA4에서 Amplitude로 이관한다고 하면 트래킹 코드가 박혀 있는 컴포넌트를 전부 뒤져서 고쳐야 한다.

## 마크업에 의도만 적으면 되게

컴포넌트는 "이 요소가 트래킹 대상이다"만 표시하고, 무엇을 어디로 보낼지는 별도 맵에서 관리하게 했다.

```html
<button data-track="banner-click">배너</button>
```

컴포넌트 코드에 GA든 Amplitude든 흔적이 없다. `data-track="banner-click"` 한 줄이 전부다.

이벤트 정의는 맵에 모아둔다.

```ts
const map = {
  'banner-click': {
    trigger: 'click',
    targets: [
      { type: 'ga', event: 'banner_click' },
      { type: 'amplitude', eventName: 'Banner Click' },
    ],
  },
};
```

"banner-click" 키 하나에 GA와 Amplitude 두 채널이 묶여 있다. 채널을 추가하거나 빼려면 이 맵만 고치면 된다.

### DOM 속성으로 동적 값 넘기기

맵에 정의된 이벤트에 런타임 데이터를 실어 보내야 할 때가 많다. 상품 ID, 캠페인명 같은 것들. 이걸 컴포넌트에서 넘기는 방법이 두 가지 있다.

개별 속성 방식:

```tsx
<div
  data-track="product-click"
  data-track-product-id={product.id}
  data-track-category={product.category}
/>
```

`data-track-` 접두사 뒤의 문자열이 파라미터 키가 된다. `data-track-product-id="123"`이면 `{ 'product-id': '123' }`으로 어댑터에 전달된다.

JSON 방식은 키가 많을 때 편하다:

```tsx
<div
  data-track="product-click"
  data-track-params={JSON.stringify({
    productId: product.id,
    category: product.category,
    price: product.price,
  })}
/>
```

둘 다 쓸 수도 있다. 같은 키가 겹치면 JSON 쪽이 이긴다:

```tsx
<div
  data-track="product-click"
  data-track-category="default"
  data-track-params={JSON.stringify({ category: 'override' })}
/>
// → dynamicParams: { category: 'override' }
```

content-view 트리거와 조합하면, 상품 카드가 화면에 보일 때 자동으로 노출 이벤트를 쏠 수 있다:

```tsx
function ProductCard({ product }: { product: Product }) {
  return (
    <div
      data-track="product-impression"
      data-track-product-id={product.id}
      data-track-params={JSON.stringify({
        productName: product.name,
        position: product.listIndex,
      })}
    >
      {/* 카드 내용 */}
    </div>
  );
}
```

맵에 이렇게 정의해두면:

```ts
'product-impression': {
  trigger: 'content-view',
  targets: [
    { type: 'ga', event: 'view_item', item_list_name: 'product_list' },
    { type: 'amplitude', eventName: 'Product Impression' },
  ],
},
```

화면에 50% 이상 노출된 시점에 두 채널로 동시에 나간다. 컴포넌트는 `IntersectionObserver`를 직접 만질 필요가 없다.

## React에 안 묶은 이유

처음에는 React 훅으로 만들까 했다. `useTracking('banner-click')` 같은 거. 근데 그러면 프레임워크가 바뀔 때 다시 만들어야 한다.

브라우저 API만 쓰기로 했다. `MutationObserver`로 DOM 변화를 감시하고, `IntersectionObserver`로 노출을 추적하고, `document.addEventListener('click', ..., true)`로 클릭을 캡처 단계에서 위임한다. React든 Vue든 결국 DOM을 만드는 건 마찬가지니까.

## MutationObserver + rAF 배칭

`MutationObserver`를 그냥 쓰면 문제가 하나 있다. React가 한 커밋 안에서 DOM을 여러 번 고치면, observer 콜백도 연달아 불린다. 매번 요소를 해석하고 핸들러에 연결하면 같은 요소를 두 번 세 번 처리하게 된다.

변경분을 `Set`에 모았다가 `requestAnimationFrame` 한 번에 처리하는 걸로 풀었다.

```ts
const pendingAdded = new Set<Element>();
const pendingRemoved = new Set<Element>();

const observer = new MutationObserver(mutations => {
  for (const mutation of mutations) {
    mergeMutationIntoPending(mutation);
  }
  scheduleMutationBatchFlush(); // rAF 한 번 예약
});
```

같은 틱에 rAF가 여러 번 잡히면 `cancelAnimationFrame`으로 이전 걸 취소하고 다시 예약한다. 결과적으로 프레임당 최대 한 번만 flush된다. `setTimeout(0)` 대신 rAF를 쓴 건, 브라우저 렌더 주기에 맞춰서 타이밍이 덜 들쑥날쑥하기 때문이다.

flush 순서는 제거 먼저, 추가 나중. IntersectionObserver 구독이 이미 빠진 노드에 남아 있으면 안 되니까.

## 클릭: 캡처 단계 리스너 하나

요소마다 리스너를 붙이는 대신, 루트에 캡처 단계 리스너 하나를 뒀다.

```ts
root.addEventListener('click', handleClick, true);

function handleClick(e: Event) {
  const target = (e.target as Element)?.closest?.('[data-track]');
  if (!target) return;
  // resolve → dispatch
}
```

캡처 단계를 쓴 건 이유가 있다. 하위 컴포넌트에서 `stopPropagation()`을 호출해도 트래킹은 이미 잡혀 있다. 실제로 모달 닫기 버튼에서 이벤트 전파를 막고 있었는데, 버블링 단계였으면 트래킹까지 같이 씹혀서 데이터가 빠졌을 거다.

## WeakSet으로 "요소당 한 번"

pageview나 content-view는 요소당 한 번만 보내야 한다. `Set` 대신 `WeakSet`을 쓴다.

```ts
const triggered = new WeakSet<Element>();

if (triggered.has(el)) continue;
triggered.add(el);
observer.unobserve(el);
```

DOM에서 빠진 요소에 대한 강한 참조가 안 남아서, SPA에서 라우트가 바뀌면 이전 요소들이 알아서 GC된다. `Set`이었으면 계속 메모리를 잡고 있었을 거다. 새 라우트에서 같은 `data-track` 키를 가진 요소가 마운트되면 새 인스턴스라 `WeakSet`에 없고, 알아서 다시 발화된다.

## 어댑터: "어디로 보낼지"만 담당

맵이 "무엇을 언제"를 정의하고, 어댑터가 "어디로"를 담당한다.

```ts
const gaAdapter = {
  type: 'ga',
  execute(target, dynamicParams) {
    gtag('event', target.event, { ...dynamicParams });
  },
};

const amplitudeAdapter = {
  type: 'amplitude',
  execute(target, dynamicParams) {
    amplitude.track(target.eventName, { ...dynamicParams });
  },
};
```

`enabled` 플래그가 있어서, 개발 환경에서 GA는 끄고 콘솔 로그만 보고 싶을 때 맵은 그대로 두고 어댑터만 꺼두면 된다.

## 타겟에 함수를 넣게 된 계기

나중에 추가한 기능인데, GA에는 `param_01` 같은 슬롯으로, Amplitude에는 `userSegment` 같은 서술적 키로 보내야 하는 상황이 있었다. 같은 데이터인데 채널마다 키 이름이 다른 거다.

```ts
'landing-view': {
  trigger: 'pageview',
  targets: [
    {
      type: 'ga',
      event: 'view_landing',
      params: (p) => ({ param_01: p.user_segment, param_02: p.coupon_state }),
    },
    {
      type: 'amplitude',
      eventName: 'Landing View',
      properties: (p) => ({ userSegment: p.user_segment, couponState: p.coupon_state }),
    },
  ],
},
```

호출하는 쪽은 어떤 채널이 붙어 있는지 몰라도 된다:

```tsx
// 선언형
<div
  data-track="landing-view"
  data-track-params={JSON.stringify({
    user_segment: 'guest_user',
    coupon_state: 'default',
  })}
/>

// 명령형
track('landing-view', { user_segment: 'guest_user', coupon_state: 'default' });
```

채널별 매핑은 맵이 알아서 한다. 함수가 있는 타겟은 dynamicParams를 어댑터에 직접 넘기지 않는다. 반환값이 이미 타겟에 합쳐져 있으니까. 이래야 GA 어댑터에 `user_segment`가 새어들어가는 걸 막을 수 있다.

## 몇 달 써보니

트래킹 리뷰가 빨라졌다. 예전에는 "이 이벤트 어디서 보내지?" 하면 컴포넌트를 뒤져야 했는데, 이제는 맵 파일 하나만 보면 된다. PM이 이벤트 이름을 바꿔달라고 하면 맵에서 한 줄 고치고 끝.

컴포넌트도 깨끗해졌다. 트래킹 때문에 `useEffect`나 `useRef`를 쓸 일이 없어졌다.

Amplitude를 추가로 붙일 때는 어댑터 하나 만들고 맵의 targets에 한 줄 추가하면 됐다. 컴포넌트는 안 건드렸다.

물론 단점도 있다. `data-track` 문자열이 맵의 키와 맞아야 하는데, 오타가 나면 에러 없이 조용히 무시된다. debug 모드에서 경고를 띄우긴 하는데, 프로덕션에서는 모른다. 타입 레벨에서 키를 강제하는 건 아직 못 했다. DOM 속성 기반이라 서버 사이드에서 동작하지 않는 것도 제약이다. 우리는 클라이언트 전용 라우트라 문제없었지만, SSR 프로젝트라면 별도 처리가 필요하다.

트래킹 시스템을 직접 만들어야 하냐고 물으면, 웬만하면 GTM으로 충분하다고 답할 것 같다. 근데 채널이 2개 이상이고, 이벤트가 수십 개를 넘어서, 컴포넌트마다 트래킹 코드가 비즈니스 로직보다 많아지는 상황이라면 한번 고려해볼 만하다. 방법이 data attribute가 아니어도 된다. 커스텀 훅이든, 디렉티브든. 트래킹이 비즈니스 로직을 잠식하지 않게 선을 긋는 게 핵심이다.

## 출처 / 참고

- [[frontend/analytics/declarative-tracking|선언적 트래킹 패턴]] — 기술 상세 위키
- `@common/tracking` 패키지 소스 코드

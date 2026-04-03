---
title: MutationObserver
aliases: [DOM Observer, DOM 변경 감지]
tags:
  - javascript
  - web-api
  - dom
  - observer
created: 2026-03-25
updated: 2026-03-25
reviewed: false
---

## 정의

DOM 트리의 변경(자식 노드 추가/제거, 속성 변경, 텍스트 변경)을 비동기적으로 감지하는 Web API. deprecated된 Mutation Events(`DOMSubtreeModified` 등)를 대체한다.

## 상세 설명

### 기본 사용법

```js
const observer = new MutationObserver((mutations) => {
  mutations.forEach(mutation => {
    console.log(mutation.type);
  });
});

observer.observe(targetNode, options);
observer.disconnect();          // 감시 중단
observer.takeRecords();         // 대기 중인 mutation 즉시 반환
```

### observe 옵션

| 옵션 | 타입 | 설명 |
|------|------|------|
| `childList` | boolean | 자식 노드 추가/제거 감지 |
| `attributes` | boolean | 속성 변경 감지 |
| `characterData` | boolean | 텍스트 내용 변경 감지 |
| `subtree` | boolean | 하위 트리 전체 감시 |
| `attributeOldValue` | boolean | 변경 전 속성값 기록 |
| `characterDataOldValue` | boolean | 변경 전 텍스트 기록 |
| `attributeFilter` | string[] | 특정 속성만 감시 |

### MutationRecord 주요 속성

| 속성 | 설명 |
|------|------|
| `type` | `childList` / `attributes` / `characterData` |
| `target` | 변경이 발생한 노드 |
| `addedNodes` / `removedNodes` | 추가/제거된 노드 NodeList |
| `attributeName` | 변경된 속성명 |
| `oldValue` | 변경 전 값 (옵션 활성화 시) |
| `previousSibling` / `nextSibling` | 추가/제거된 노드의 형제 |

### 동작 원리

콜백은 동기적으로 매번 호출되지 않고, **마이크로태스크 큐**에서 변경사항을 배치로 모아 한 번에 실행된다. 이 덕분에 Mutation Events 대비 성능이 크게 향상된다.

### 성능 주의점

- `subtree: true`는 대규모 DOM에서 비용이 클 수 있음
- 콜백 내에서 DOM을 변경하면 추가 mutation이 발생할 수 있으므로 주의

## 실무 적용

### 동적 요소 감지 (SPA, 광고 차단 등)

```js
const observer = new MutationObserver(mutations => {
  for (const m of mutations) {
    for (const node of m.addedNodes) {
      if (node.nodeType === 1 && node.matches('.ad-banner')) {
        node.remove();
      }
    }
  }
});
observer.observe(document.body, { childList: true, subtree: true });
```

### 속성 변경 감지 (테마 전환)

```js
const observer = new MutationObserver(() => {
  const theme = document.documentElement.getAttribute('data-theme');
  console.log('테마 변경:', theme);
});
observer.observe(document.documentElement, {
  attributes: true,
  attributeFilter: ['data-theme'],
});
```

### Observer 패밀리 비교

| Observer | 용도 |
|----------|------|
| **MutationObserver** | DOM 구조/속성/텍스트 변경 |
| **ResizeObserver** | 요소 크기 변경 |
| **IntersectionObserver** | 뷰포트 교차(가시성) |

## 출처 / 참고

- [MDN - MutationObserver](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver)
- [[declarative-tracking|Declarative Tracking - 선언적 트래킹 패턴]] — MutationObserver를 활용한 트래킹 구현 사례

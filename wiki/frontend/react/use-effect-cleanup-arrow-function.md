---
title: useEffect cleanup과 화살표 함수 패턴
aliases:
  - useEffect return
  - useEffect cleanup
tags:
  - react
  - hooks
  - use-effect
  - cleanup
  - arrow-function
created: 2026-03-26
updated: 2026-03-26
reviewed: false
---

## 정의

`useEffect`의 콜백이 함수를 반환하면, React가 컴포넌트 언마운트(또는 deps 변경) 시 해당 함수를 cleanup으로 호출하는 패턴. 화살표 함수의 암묵적 반환과 결합하면 초기화+정리를 한 줄로 표현할 수 있다.

## 상세 설명

### useEffect의 cleanup 메커니즘

```ts
useEffect(() => {
  // 마운트 시 실행
  const handler = () => console.log('click');
  document.addEventListener('click', handler);

  // return한 함수 = 언마운트 시 React가 호출
  return () => {
    document.removeEventListener('click', handler);
  };
}, []);
```

- 마운트 → 콜백 본문 실행 (리스너 등록 등)
- 언마운트 → return된 함수 실행 (리스너 해제 등)
- deps 변경 시 → 이전 cleanup 실행 → 새 콜백 실행

### 화살표 함수의 암묵적 반환 활용

init 함수가 cleanup 함수를 반환하는 구조라면:

```ts
function init() {
  document.addEventListener('click', handleClick);
  observer.observe(root, { childList: true, subtree: true });

  return () => {
    document.removeEventListener('click', handleClick);
    observer.disconnect();
  };
}
```

useEffect에서 한 줄로 사용할 수 있다:

```ts
// 화살표 함수가 init()의 반환값을 암묵적으로 return
useEffect(() => init(), []);

// 위 코드는 아래와 완전히 동일
useEffect(() => {
  const cleanup = init();
  return cleanup;
}, []);
```

**핵심**: `() => init()`에서 `init()`은 **호출**(본문 실행)이고, 그 **반환값**(cleanup 함수)이 화살표 함수에 의해 암묵적으로 return되어 React에 전달된다.

### 타임라인

1. 컴포넌트 마운트 → `init()` 본문 실행 → 리소스 등록
2. 컴포넌트 언마운트 → React가 return된 cleanup 호출 → 리소스 해제

### 주의: void 반환과의 차이

cleanup이 필요 없는 경우 함수 반환값이 없어야 한다. 의도치 않게 함수를 반환하면 React가 cleanup으로 호출해버린다.

```ts
// 나쁜 예 — fetch가 Promise를 반환하므로 React에 경고 발생
useEffect(() => fetch('/api/data'), []);

// 좋은 예 — 중괄호로 감싸서 반환값 차단
useEffect(() => { fetch('/api/data'); }, []);
```

### React 18 StrictMode

개발 모드에서 `useEffect`가 mount → unmount → mount로 2번 실행된다. cleanup이 모든 리소스를 올바르게 해제하면 문제 없다.

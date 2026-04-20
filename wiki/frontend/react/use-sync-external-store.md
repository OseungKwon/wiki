---
title: useSyncExternalStore
aliases: [useSyncExternalStore, useExternalStore]
tags:
  - react
  - hooks
  - concurrent
  - external-store
created: 2026-04-20
updated: 2026-04-20
reviewed: false
---

## 정의

React 18에서 도입된 훅으로, 외부 스토어(external store)를 Concurrent Rendering 환경에서 안전하게 구독하기 위한 API.

## 상세 설명

### 핵심 문제: Tearing

Concurrent Rendering에서는 렌더링이 중단/재개될 수 있다. 이 과정에서 외부 스토어 값이 변경되면, 같은 렌더 트리 안에서 컴포넌트마다 다른 값을 읽는 **tearing(찢어짐)** 이 발생한다.

```
컴포넌트 A: "old" 읽음    ─── t1 (렌더 시작)
스토어 → "new" 변경       ─── t2 (렌더 도중)
컴포넌트 B: "new" 읽음    ─── t3
→ A="old", B="new" — tearing 발생
```

### API 시그니처

```tsx
const value = useSyncExternalStore(
  subscribe,        // (callback) => unsubscribe — 구독 함수
  getSnapshot,      // () => value — 클라이언트 스냅샷
  getServerSnapshot // () => value — SSR 스냅샷 (선택)
);
```

### useState + useEffect로 부족한 이유

```tsx
// 문제가 있는 패턴
function useStore(store) {
  const [value, setValue] = useState(store.getValue());
  useEffect(() => {
    return store.subscribe(() => setValue(store.getValue()));
  }, [store]);
  return value;
}
```

| 문제 | 원인 |
|------|------|
| Stale value | useEffect는 커밋 후 실행 → 마운트~구독 등록 사이에 값이 바뀌면 놓침 |
| Tearing | Concurrent 렌더링 중 컴포넌트마다 다른 시점의 값을 읽음 |
| SSR 에러 | 서버에서 외부 API(localStorage 등)가 없어서 크래시 |

### useSyncExternalStore가 해결하는 것

1. **Tearing 방지** — 렌더 중 스토어 변경 감지 시 동기적으로 재렌더링
2. **Stale value 방지** — 구독 등록과 값 읽기가 렌더와 동기적으로 묶임
3. **SSR 지원** — `getServerSnapshot`으로 서버 렌더링 시 별도 값 제공
4. **자동 최적화** — 스냅샷이 동일하면(`Object.is`) 불필요한 리렌더 방지

```
[useState + useEffect]
렌더 → 커밋 → useEffect → 구독 등록 (이 갭에서 변경 놓침)

[useSyncExternalStore]
구독 등록 + 스냅샷 읽기가 렌더와 동기적으로 묶임 → 갭 없음
```

## 실무 적용: localStorage 비교

### Before — useState + useEffect

```tsx
function useLocalStorage(key: string) {
  const [value, setValue] = useState(() => localStorage.getItem(key));

  useEffect(() => {
    const handler = (e: StorageEvent) => {
      if (e.key === key) setValue(e.newValue);
    };
    window.addEventListener('storage', handler);
    return () => window.removeEventListener('storage', handler);
  }, [key]);

  return value;
}
```

**문제점:**
- `StorageEvent`는 **다른 탭**에서 변경했을 때만 발생 — 같은 탭 변경 감지 불가
- 마운트~구독 등록 사이 갭에서 stale value 발생 가능
- SSR 환경에서 `localStorage` 미존재로 크래시

### After — useSyncExternalStore

```tsx
function useLocalStorage(key: string) {
  const subscribe = useCallback(
    (callback: () => void) => {
      const handleStorage = (e: StorageEvent) => {
        if (e.key === key) callback();
      };
      window.addEventListener('storage', handleStorage);

      // 같은 탭 변경 감지 (커스텀 이벤트)
      const handleLocal = (e: CustomEvent) => {
        if (e.detail === key) callback();
      };
      window.addEventListener('local-storage', handleLocal as EventListener);

      return () => {
        window.removeEventListener('storage', handleStorage);
        window.removeEventListener('local-storage', handleLocal as EventListener);
      };
    },
    [key],
  );

  const getSnapshot = useCallback(() => localStorage.getItem(key), [key]);
  const getServerSnapshot = useCallback(() => null, []);

  return useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
}

// 같은 탭에서도 동기화되는 setter
function setLocalStorage(key: string, value: string) {
  localStorage.setItem(key, value);
  window.dispatchEvent(new CustomEvent('local-storage', { detail: key }));
}
```

### 사용 판단 기준

| 사용 O | 사용 X |
|--------|--------|
| Redux, Zustand 등 외부 상태 라이브러리 | React state (useState, useReducer) |
| 브라우저 API (localStorage, matchMedia) | Props, Context |
| 직접 만든 pub/sub 스토어 | Server Component의 데이터 |

## 출처 / 참고

- [React 공식 문서 — useSyncExternalStore](https://react.dev/reference/react/useSyncExternalStore)
- [[fiber-wip-tree]] — Concurrent Rendering 동작 원리
- [[rendering-lifecycle-with-suspense]] — React 렌더링 생애주기

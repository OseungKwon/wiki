---
title: "Server Components vs 클라이언트 서버 상태관리"
aliases: [RSC vs TanStack Query, 서버 컴포넌트 상태관리]
tags:
  - react
  - rsc
  - server-components
  - tanstack-query
  - state-management
created: 2026-05-04
updated: 2026-05-04
reviewed: false
---

## 정의
React Server Components(RSC)가 서버 데이터의 **초기 로딩**을 단순화했지만, 클라이언트 서버 상태관리(TanStack Query 등)는 **인터랙션 이후의 데이터 관리** 영역에서 여전히 유효하다. 둘은 경쟁이 아닌 **역할 분담** 관계다.

## 상세 설명

### RSC가 해결하는 것
- 서버에서 직접 `await fetch()` → HTML로 전달
- queryKey, staleTime, loading state 관리가 불필요
- 서버에서 병렬 fetch로 waterfall 해소

```tsx
// Server Component — 별도 상태관리 라이브러리 불필요
async function UserProfile({ userId }: { userId: string }) {
  const user = await getUser(userId);
  return <div>{user.name}</div>;
}
```

### RSC가 해결하지 못하는 것

서버 컴포넌트는 **초기 렌더링 시점**에만 동작한다. 이후 클라이언트 인터랙션은 여전히 클라이언트 영역:

| 시나리오 | RSC | TanStack Query |
|---------|-----|---------------|
| 초기 페이지 데이터 로딩 | O | 불필요 |
| 정적/준정적 콘텐츠 | O | 불필요 |
| 검색어 입력 → 실시간 결과 | X | O |
| 무한 스크롤 | X | O |
| 폴링 (실시간 알림, 주문 상태) | X | O |
| Optimistic update (좋아요, 장바구니) | X | O |
| 오프라인 지원 | X | O |
| 세밀한 캐시 조작 (특정 아이템만 갱신) | X | O |

### Mutation 관점

RSC에는 mutation 개념이 없다. `revalidatePath`/`revalidateTag`로 서버 캐시를 날릴 수는 있지만:
- optimistic update 불가 — 서버 응답까지 UI 대기
- 세밀한 캐시 조작 불가
- 오프라인 지원 불가

### 클라이언트 전용 SPA의 경우

React Router v7 `clientLoader` 기반처럼 사실상 SPA인 앱에서는 RSC를 사용하지 않으므로, TanStack Query의 가치가 그대로 유효하다.

### 판단 기준 요약

- **읽기(Read)만** 필요하고 인터랙션 없음 → Server Component로 충분
- **쓰기(Write), 실시간성, 캐시 동기화** 필요 → TanStack Query 여전히 유효
- **하이브리드**: 초기 데이터는 RSC, 이후 인터랙션은 TanStack Query로 hydrate하여 이어받는 패턴도 가능

## 출처 / 참고
- [[rsc-async-optimization|RSC 비동기 최적화]]
- [[query-options-factory-pattern|TanStack Query queryOptions 팩토리 패턴]]

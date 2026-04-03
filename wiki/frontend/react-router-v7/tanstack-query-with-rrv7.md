---
title: TanStack Query와 React Router v7 조합
aliases:
  - RRv7 TanStack Query
  - react-query react-router
tags:
  - react-router-v7
  - tanstack-query
  - data-fetching
  - caching
created: 2026-03-18
updated: 2026-03-18
reviewed: false
---

## 정의
React Router v7의 loader/action만으로 대부분의 데이터 흐름은 충분하며, TanStack Query는 loader 재검증 모델로 부족한 특수 케이스에서 보완적으로 사용한다.

## 상세 설명

### TanStack Query가 필요한 경우
| 상황 | 이유 |
|---|---|
| **폴링 / 실시간 갱신** | `refetchInterval` 등 자동 재요청. loader는 URL 변경/action 후에만 재실행 |
| **무한 스크롤** | `useInfiniteQuery`의 페이지 누적 관리 |
| **글로벌 공유 데이터** | 사용자 정보, 알림 등 여러 라우트에서 공유하는 데이터 |
| **세밀한 캐시 제어** | `staleTime`, `gcTime` 등 캐시 전략 세밀 조정 |
| **Optimistic Update** | action 재검증보다 즉각적인 UI 반영 필요 시 |
| **조건부 쿼리** | `enabled` 옵션으로 특정 조건에서만 실행 |

### loader/action으로 충분한 경우
- 페이지 진입 시 한 번 데이터 로드
- 폼 제출 후 페이지 갱신
- 단순 CRUD
- 네비게이션 기반 데이터 흐름

### 함께 쓰는 패턴
loader에서 초기 데이터를 보장하고, TanStack Query가 이후 캐시/재검증을 관리한다.

```typescript
export const clientLoader = async () => {
  const queryClient = getQueryClient();
  await queryClient.ensureQueryData(productsQueryOptions());
  return {};
};

export default function Products() {
  const { data } = useQuery(productsQueryOptions());
  return <ProductList items={data} />;
}
```

`ensureQueryData`는 캐시가 있으면 캐시를 반환하고, 없으면 fetch한다. 이렇게 하면 loader가 데이터 로딩 시점을 보장하면서 TanStack Query의 캐시/폴링 등 고급 기능을 활용할 수 있다.

## 출처 / 참고
- [TanStack Query 공식 문서](https://tanstack.com/query)
- [[loader-action|React Router v7 Loader & Action]]

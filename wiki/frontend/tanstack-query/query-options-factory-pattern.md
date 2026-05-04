---
title: "TanStack Query - queryOptions 팩토리 패턴"
aliases: [query factory, queryOptions pattern, 쿼리 팩토리]
tags:
  - tanstack-query
  - react-query
  - design-pattern
  - typescript
created: 2026-05-04
updated: 2026-05-04
reviewed: false
---

## 정의
`queryOptions`는 TanStack Query v5에서 도입된 헬퍼로, queryKey·queryFn·staleTime 등 쿼리 설정을 하나의 객체로 묶어 재사용할 수 있게 한다. 이를 도메인별 객체로 구조화한 것이 **쿼리 팩토리 패턴**이다.

## 상세 설명

### 기존 방식의 문제
- queryKey 불일치: 개발자마다 `['user']` vs `['users']` 등 다른 키 사용
- 설정 중복: 동일 데이터에 다른 staleTime 적용
- 타입 안전성 부족: invalidate 시 오타 위험
- 프리페칭 시 전체 설정을 반복 작성

### queryOptions 팩토리

도메인 단위로 쿼리 설정을 객체로 묶는다:

```typescript
export const userQueries = {
  all: () => ['users'] as const,
  lists: () => [...userQueries.all(), 'list'] as const,
  list: (filters?: UserFilters) =>
    queryOptions({
      queryKey: [...userQueries.lists(), filters],
      queryFn: () => fetchUsers(filters),
      staleTime: 5 * 60 * 1000,
    }),
  details: () => [...userQueries.all(), 'detail'] as const,
  detail: (userId: string) =>
    queryOptions({
      queryKey: [...userQueries.details(), userId],
      queryFn: () => fetchUser(userId),
      staleTime: 5 * 60 * 1000,
    }),
};
```

### 사용처별 활용

```typescript
// 컴포넌트
useQuery(userQueries.detail(userId));

// 프리페칭
queryClient.prefetchQuery(userQueries.detail(userId));

// 캐시 무효화 — 계층적으로 가능
queryClient.invalidateQueries({ queryKey: userQueries.all() });     // 전체
queryClient.invalidateQueries({ queryKey: userQueries.details() }); // 상세만
queryClient.invalidateQueries(userQueries.detail(userId));          // 특정 유저
```

### 계층적 키 설계

`all → lists → list`, `all → details → detail` 구조로 무효화 범위를 세밀하게 제어할 수 있다.

### Mutation 통합 (optimistic update)

```typescript
export function useUpdateUser() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: ({ userId, updates }) => updateUser(userId, updates),
    onMutate: async ({ userId, updates }) => {
      await queryClient.cancelQueries(userQueries.detail(userId));
      const previous = queryClient.getQueryData(userQueries.detail(userId).queryKey);
      queryClient.setQueryData(
        userQueries.detail(userId).queryKey,
        (old: User) => ({ ...old, ...updates })
      );
      return { previous };
    },
    onError: (err, { userId }, context) => {
      queryClient.setQueryData(userQueries.detail(userId).queryKey, context?.previous);
    },
  });
}
```

### 도메인 중심 파일 구조

```
api/post/
├── axios.ts     # fetch 함수
├── queries.ts   # queryOptions 객체 (postQueries)
└── types.ts     # 타입 정의
```

mutation 후 invalidate할 queryKey를 같은 도메인 파일에서 바로 찾을 수 있어 파일 횡단이 줄어든다.

### 주의사항
- `as const` 누락 시 queryKey 타입이 `string[]`로 넓어짐
- 팩토리와 인라인 설정을 혼용하면 단일 소스 원칙이 깨짐
- 키 계층을 과도하게 깊게 설계하지 말 것

## 출처 / 참고
- [Why queryOptions Will Change How You Use TanStack Query](https://medium.com/@jmytwenty8/why-queryoptions-will-change-how-you-use-tanstack-query-141608dd5c3c)
- [React Query Options 기반 관리 패턴](https://velog.io/@ubin_ing/react-query-options-basement-pattern)
- [[typescript-options-utility-types|TanStack Query TypeScript 유틸리티 타입]]
- [[tanstack-query-with-rrv7|TanStack Query와 React Router v7 조합]]

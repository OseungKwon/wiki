---
title: "TanStack Query - TypeScript Options 유틸리티 타입"
aliases:
  - QueryOptions 타입
  - MutationOptions 타입
tags:
  - tanstack-query
  - typescript
  - type-inference
  - react-query
created: 2026-04-30
updated: 2026-04-30
reviewed: false
---

## 정의

커스텀 훅이 외부에서 받을 options 파라미터의 타입을 API 함수 시그니처로부터 자동 추론하는 유틸리티 타입 패턴. `queryKey`/`queryFn`/`mutationFn` 등 훅 내부에서 결정되는 옵션은 `Omit`으로 제거하고, `enabled`, `select`, `staleTime`, `onSuccess` 같은 동작 옵션만 외부에 열어준다.

## 상세 설명

### 타입 정의

```typescript
type QueryOptions<TFn extends (...args: any) => any> = Omit<
  UseQueryOptions<Awaited<ReturnType<TFn>>, Error>,
  'queryKey' | 'queryFn'
>;

type MutationOptions<TFn extends (...args: any) => any> = Omit<
  UseMutationOptions<Awaited<ReturnType<TFn>>, Error, Parameters<TFn>[0], unknown>,
  'mutationFn'
>;
```

- `ReturnType<TFn>` — API 함수의 반환 타입에서 data 타입 추론
- `Parameters<TFn>[0]` — API 함수의 첫 번째 파라미터에서 variables 타입 추론
- `Awaited` — Promise unwrap 처리

### 사용 패턴

```typescript
// API 함수
const fetchGroups = async (id: number): Promise<Group[]> => { ... };
const createGroup = async (payload: CreateGroupPayload): Promise<Group> => { ... };

// Query 훅
const useGroupQuery = (
  id: number,
  options?: QueryOptions<typeof fetchGroups>,
) => {
  return useQuery({
    queryKey: ['groups', id],
    queryFn: () => fetchGroups(id),
    ...options,
  });
};

// Mutation 훅
const useCreateGroupMutation = (
  options?: MutationOptions<typeof createGroup>,
) => {
  return useMutation({
    mutationFn: createGroup,
    ...options,
  });
};

// 컴포넌트에서 사용
const { data } = useGroupQuery(id, {
  enabled: !!id,
  staleTime: 10_000,
  select: (data) => data.filter((g) => g.active), // data: Group[] 추론
});

const { mutate } = useCreateGroupMutation({
  onSuccess: (data) => { ... }, // data: Group 추론
});
```

### 공식 방식과의 비교

TanStack Query v5는 `queryOptions()` / `mutationOptions()` 헬퍼 함수를 공식 제공한다.

| 관점 | 공식 (`queryOptions()`) | 커스텀 (`QueryOptions<TFn>`) |
|---|---|---|
| 목적 | 옵션 객체를 여러 곳에서 재사용 | 커스텀 훅의 options 파라미터 타입 정의 |
| 훅 옵션 타입 | `Omit<ReturnType<typeof xxxOptions>, ...>` 매번 작성 | `QueryOptions<typeof fetchFn>` 한 줄 |
| 보일러플레이트 | 옵션 팩토리 함수 + 훅 = 2개 | 훅 하나로 완결 |
| queryKey 공유 | `options.queryKey`로 추출 가능 | 훅 안에 캡슐화됨 |
| 훅 외부 재사용 | `prefetchQuery`, `setQueryData` 등 가능 | 불가 |
| 런타임 코드 | identity 함수 호출 | 타입만 존재 |

"커스텀 훅에 options props 넘기기" 패턴에 한정하면 커스텀 방식이 더 간결하다. 공식 방식은 훅 바깥에서도 옵션을 공유해야 할 때(prefetch, invalidation 등) 진가를 발휘한다. 둘은 경쟁이 아니라 보완 관계.

### TanStack Query의 TypeScript 철학

공식 문서에서 명시하는 원칙:

> "TypeScript works best if you let it infer what type something should be on its own, which makes code easier to write and read, and in many instances can make code look exactly like JavaScript."

- 제네릭을 직접 쓰지 않는다 — `useQuery<Group[], Error>(...)` 지양
- `queryFn`의 반환 타입만 잘 정의하면 나머지는 전부 추론
- TypeScript 코드가 JavaScript처럼 보일수록 좋다

커스텀 `QueryOptions<TFn>` 타입도 이 철학에 부합한다. API 함수 하나만 넘기면 모든 타입이 자동 추론되므로 사용 측에서 제네릭이나 타입 어노테이션을 작성할 필요가 없다.

## 출처 / 참고

- [TanStack Query - TypeScript Guide](https://tanstack.com/query/latest/docs/framework/react/typescript)
- [TanStack Query - Query Options Guide](https://tanstack.com/query/v5/docs/react/guides/query-options)
- [TanStack Query - mutationOptions Reference](https://tanstack.com/query/v5/docs/framework/react/reference/mutationOptions)
- [TkDodo - React Query and TypeScript](https://tkdodo.eu/blog/react-query-and-type-script)
- [TkDodo - The Query Options API](https://tkdodo.eu/blog/the-query-options-api)
- [[tanstack-query-with-rrv7|TanStack Query와 React Router v7 조합]]

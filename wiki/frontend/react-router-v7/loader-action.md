---
title: React Router v7 - Loader & Action
aliases:
  - RRv7 loader
  - RRv7 action
  - clientLoader
tags:
  - react-router-v7
  - loader
  - action
  - data-fetching
created: 2026-03-18
updated: 2026-03-18
reviewed: false
---

## 정의
React Router v7에서 **loader**는 라우트 렌더링 전 데이터를 로드하는 함수이고, **action**은 폼 제출 등 데이터 변경(mutation)을 처리하는 함수이다.

## 상세 설명

### 데이터 흐름 (핵심 아키텍처)
URL이 source of truth이며, 데이터는 라우트 단위로 관리된다.

```
URL 변경 → loader (데이터 로드) → 컴포넌트 렌더링
사용자 액션 → action (데이터 변경) → loader 재검증 → UI 갱신
```

### Loader
- **GET 요청 시 실행** — 페이지 진입, 링크 클릭, 브라우저 뒤로가기
- **병렬 실행** — 중첩 라우트의 loader들이 동시에 실행 (워터폴 방지)
- **자동 재검증** — action 완료 후 해당 페이지의 loader가 자동 재실행

```typescript
export async function loader({ request, params }: LoaderFunctionArgs) {
  const products = await fetchProducts();
  return { products };
}

export default function Products() {
  const { products } = useLoaderData<typeof loader>();
  return <ProductList items={products} />;
}
```

### loader vs clientLoader
| | `loader` (서버) | `clientLoader` (클라이언트) |
|---|---|---|
| 실행 환경 | 서버 (SSR) | 브라우저 |
| 사용 시점 | SSR이 필요한 페이지 | client-only, 인증 필요 시 |

`clientLoader`를 export하면 해당 라우트는 client-only route가 된다 (SSR 없음).

### Action
- **non-GET 요청 시 실행** — POST, PUT, DELETE
- `<Form>`, `useSubmit`, `useFetcher`로 호출
- action 완료 후 loader가 자동 재실행되어 데이터 동기화

```typescript
export async function action({ request }: ActionFunctionArgs) {
  const formData = await request.formData();
  await createProduct({ name: formData.get("name") });
  return redirect("/products");
}
```

`clientAction`은 클라이언트 전용 버전이다.

### 주요 훅
| 훅 | 용도 |
|---|---|
| `useLoaderData()` | loader 데이터 접근 |
| `useActionData()` | action 반환값 접근 |
| `useNavigation()` | 로딩/제출 상태 (pending UI) |
| `useFetcher()` | 네비게이션 없이 loader/action 호출 |
| `useSubmit()` | 프로그래밍적 폼 제출 |

### useFetcher
페이지 이동 없이 데이터 로드/mutation 실행 시 사용. 가장 유연한 도구.

```typescript
function LikeButton({ productId }) {
  const fetcher = useFetcher();
  return (
    <fetcher.Form method="post" action="/api/like">
      <input type="hidden" name="id" value={productId} />
      <button disabled={fetcher.state !== "idle"}>좋아요</button>
    </fetcher.Form>
  );
}
```

## 출처 / 참고
- [React Router v7 공식 문서](https://reactrouter.com)
- [[tanstack-query-with-rrv7|TanStack Query와 RRv7 조합]]

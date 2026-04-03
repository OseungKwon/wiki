---
title: React 19 use API
aliases:
  - use hook
  - React use
tags:
  - react
  - use
  - suspense
  - data-fetching
  - react-19
created: 2026-04-03
updated: 2026-04-03
reviewed: false
---

## 정의
`use`는 React 19에서 도입된 API로, **Promise 또는 Context를 읽는다**. 다른 hook과 달리 조건문/반복문 안에서 호출할 수 있으며, 미완료 Promise는 가장 가까운 Suspense boundary를 트리거한다.

## 상세 설명

### 두 가지 용도

#### 1. Promise 읽기
```tsx
function UserProfile({ userPromise }) {
  const user = use(userPromise); // 미완료 → suspend, resolve → 값 반환
  return <div>{user.name}</div>;
}

// 부모에서 Promise를 전달
function Page({ id }) {
  const userPromise = fetchUser(id);
  return (
    <Suspense fallback={<Skeleton />}>
      <UserProfile userPromise={userPromise} />
    </Suspense>
  );
}
```

- 미완료 Promise → Suspense fallback 표시
- resolve → 값 반환, 정상 렌더링
- reject → 가장 가까운 Error Boundary로 전파

#### 2. Context 읽기 (useContext 대체)
```tsx
function Button() {
  if (isDarkMode) {
    const theme = use(ThemeContext); // 조건문 안에서 가능
    return <button style={{ color: theme.primary }}>Click</button>;
  }
  return <button>Click</button>;
}
```

### 일반 hook과의 차이

| | `use` | 일반 hooks |
|--|--|--|
| 조건문/반복문 | 호출 가능 | 불가 (Rules of Hooks) |
| 받는 값 | Promise 또는 Context | - |
| early return 이후 | 호출 가능 | 불가 |

### 핵심 규칙: Promise는 렌더 외부에서 생성

```tsx
// ❌ 매 렌더마다 새 Promise → 무한 suspend
function Bad({ id }) {
  const user = use(fetch(`/api/users/${id}`).then(r => r.json()));
}

// ✅ 부모/Server Component/라우터에서 전달
function Parent({ id }) {
  const userPromise = fetchUser(id); // 캐시되거나 안정적 참조
  return (
    <Suspense fallback={<Loading />}>
      <Child userPromise={userPromise} />
    </Suspense>
  );
}
```

### Best Practice: GET vs POST 구분

`use`는 **데이터 읽기(GET) 전용**으로 설계되었다.

**이유:**
1. Suspense 연동은 "데이터 준비까지 대기"라는 읽기 시나리오
2. React는 렌더를 중단/재시도할 수 있어, 부수효과가 있으면 중복 실행 위험
3. Concurrent 모드에서 같은 컴포넌트가 여러 번 렌더링될 수 있음

```tsx
// ✅ use에 적합 — 읽기
use(fetchUserProfile(id))
use(fetchProducts(category))

// ❌ use에 부적합 — 부수효과
use(createOrder(cart))      // POST
use(updateProfile(data))    // PUT
use(deleteComment(id))      // DELETE
```

### Mutation은 전용 API 사용

| 시나리오 | API | 비고 |
|----------|-----|------|
| 데이터 읽기 | `use(promise)` | Suspense 필수 |
| form 제출 (POST/PUT/DELETE) | `useActionState` + `<form action>` | 에러/pending 내장 |
| 이벤트 핸들러에서 mutation | `useTransition` + async 함수 | isPending 제공 |
| 낙관적 업데이트 | `useOptimistic` | mutation 중 즉시 UI 반영 |

#### useActionState (form mutation)
```tsx
function OrderForm() {
  const [state, submitAction, isPending] = useActionState(
    async (prevState, formData) => {
      const result = await createOrder(formData);
      if (result.error) return { error: result.error };
      redirect('/orders/' + result.id);
    },
    { error: null }
  );

  return (
    <form action={submitAction}>
      <input name="product" />
      <button disabled={isPending}>
        {isPending ? '주문 중...' : '주문하기'}
      </button>
      {state.error && <p>{state.error}</p>}
    </form>
  );
}
```

#### useTransition (이벤트 핸들러 mutation)
```tsx
function LikeButton({ postId }) {
  const [isPending, startTransition] = useTransition();
  const [liked, setLiked] = useState(false);

  function handleLike() {
    startTransition(async () => {
      await likePost(postId);
      setLiked(true);
    });
  }

  return (
    <button onClick={handleLike} disabled={isPending}>
      {liked ? '❤️' : '🤍'}
    </button>
  );
}
```

#### 읽기 + 쓰기 조합 예시
```tsx
function Comments({ commentsPromise, postId }) {
  const comments = use(commentsPromise);                   // 읽기
  const [optimistic, addOptimistic] = useOptimistic(comments);

  const [, submitAction] = useActionState(async (prev, formData) => {
    addOptimistic([...optimistic, { text: formData.get('text'), pending: true }]);
    await postComment(postId, formData.get('text'));        // 쓰기
  }, null);

  return (
    <>
      <ul>
        {optimistic.map((c, i) => (
          <li key={i} style={{ opacity: c.pending ? 0.5 : 1 }}>{c.text}</li>
        ))}
      </ul>
      <form action={submitAction}>
        <input name="text" />
        <button>댓글 작성</button>
      </form>
    </>
  );
}
```

**원칙: `use`는 읽기, Action(`useActionState`/`useTransition`)은 쓰기. React 19는 이 둘을 명확히 분리했다.**

## 출처 / 참고
- [React 공식 문서 — use](https://react.dev/reference/react/use)
- [[suspense-version-diff|React Suspense 버전별 동작 차이]] — `use()` 도입 배경
- [[rendering-lifecycle-with-suspense|React 렌더링 생애주기]] — Suspense 내부 동작
- [[fiber-wip-tree|Fiber WIP 트리]] — 렌더 중단/재시도가 가능한 이유

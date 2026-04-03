---
title: 선언형 프로그래밍
aliases:
  - Declarative Programming
  - 선언적 프로그래밍
  - What vs How
tags:
  - programming-paradigm
  - declarative
  - imperative
created: 2026-04-03
updated: 2026-04-03
---

## 정의
**무엇(What)**을 원하는지 기술하고, **어떻게(How)** 할지는 런타임/프레임워크에 위임하는 프로그래밍 패러다임. 명령형(Imperative)과 대비되며, 본질적으로는 명령형 코드 위에 추상화 계층을 씌운 것이다.

## 상세 설명

### 핵심 구분

| | 선언형 (Declarative) | 명령형 (Imperative) |
|--|--|--|
| 관심사 | **무엇**을 원하는지 기술 | **어떻게** 할지 절차를 기술 |
| 제어 흐름 | 런타임/프레임워크에 위임 | 개발자가 직접 제어 |
| 상태 관리 | 결과 상태를 선언 | 상태 변경 단계를 직접 기술 |

### 같은 문제, 두 가지 접근

명령형 — 배열에서 짝수만 뽑기:
```ts
const result = [];
for (let i = 0; i < nums.length; i++) {
  if (nums[i] % 2 === 0) result.push(nums[i]);
}
```

선언형:
```ts
const result = nums.filter(n => n % 2 === 0);
```

### UI에서의 선언형

명령형 (jQuery):
```js
btn.addEventListener('click', () => {
  count++;
  counter.textContent = count;
  if (count > 10) counter.style.color = 'red';
});
```

선언형 (React):
```tsx
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <span style={{ color: count > 10 ? 'red' : 'black' }}>
      {count}
    </span>
  );
}
```

"count가 이 값일 때 UI는 이렇게 보여야 한다" — 결과 상태를 선언하고 DOM 조작은 React에 위임.

### 추상화 계층으로서의 선언형

```
선언형 코드 (What)
    ↓
추상화 계층 (프레임워크/런타임이 How를 처리)
    ↓
명령형 실행 (CPU 명령은 언제나 명령형)
```

| 선언형 코드 | 추상화 계층 |
|------------|------------|
| SQL `SELECT * WHERE age > 20` | DB 엔진이 인덱스/풀 스캔 결정 |
| React JSX `<List items={data} />` | Reconciler가 diff 후 최소 DOM 변경 |
| CSS `display: flex` | 브라우저 레이아웃 엔진이 좌표 계산 |
| HTML `<form>` | 브라우저가 submit, 유효성 검증 처리 |

### 선호되는 이유

- **가독성**: 의도(What)가 코드에 바로 드러남
- **유지보수**: How를 바꿔도 선언부는 안정적
- **최적화 위임**: 런타임이 실행 방법을 최적화 (React batching, SQL 쿼리 플래닝)
- **예측 가능성**: 같은 입력 → 같은 선언 → 같은 출력

### 흔한 오해

- "선언형 = 함수형"이 아님 — SQL은 선언형이지만 함수형 언어는 아님
- "선언형이 항상 좋다"가 아님 — 세밀한 제어가 필요하면 명령형이 적합 (애니메이션, 저수준 최적화)
- 대부분의 실제 코드는 **혼합** — React 안에서도 `useEffect`는 명령형 escape hatch

## 출처 / 참고
- [[declarative-tracking|선언적 트래킹 패턴]] — 선언형 원칙의 실무 적용 사례

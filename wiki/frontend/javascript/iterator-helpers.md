---
title: JavaScript Iterator Helpers
aliases: [이터레이터 헬퍼, Iterator Helpers]
tags:
  - javascript
  - iterator
  - lazy-evaluation
  - ecmascript-2025
created: 2026-03-20
updated: 2026-03-20
reviewed: false
---

## 정의

Iterator 프로토타입에 추가된 체이너블 메서드들로, 배열 메서드와 달리 **lazy evaluation**으로 동작하여 필요한 만큼만 처리한다. ECMAScript 2025에 정식 포함.

## 상세 설명

### 핵심 문제

배열 메서드 체이닝은 각 단계마다 중간 배열을 생성하고, 결과의 일부만 필요해도 전체를 처리한다.

```javascript
// ❌ 중간 배열 3개 생성, 전체 아이템 처리
const result = items
  .filter(isVisible)
  .map(transform)
  .slice(0, 10);
```

### Iterator Helpers 방식

```javascript
// ✅ 필요한 10개만 처리, 중간 배열 없음
const result = items
  .values()
  .filter(isVisible)
  .map(transform)
  .take(10)
  .toArray();
```

`.values()`로 Iterator를 획득한 뒤 체이닝. 각 메서드는 lazy하게 동작하며 `.toArray()`나 `for...of`로 소비할 때 실행된다.

### 사용 가능한 메서드

| 메서드 | 설명 | 반환 |
|--------|------|------|
| `.map(fn)` | 변환 | Iterator |
| `.filter(fn)` | 필터링 | Iterator |
| `.flatMap(fn)` | 평탄화 + 변환 | Iterator |
| `.take(n)` | 앞에서 n개 | Iterator |
| `.drop(n)` | 앞에서 n개 건너뛰기 | Iterator |
| `.find(fn)` | 조건 만족하는 첫 값 | value |
| `.some(fn)` / `.every(fn)` | 조건 검사 | boolean |
| `.reduce(fn, init)` | 누적 | value |
| `.toArray()` | 배열 변환 | Array |

### 주의사항

- **일회성**: Iterator는 소비하면 끝. 재사용 불가
- **Lazy 실행**: 소비하기 전까지 부수효과가 실행되지 않음
- **랜덤 접근 불가**: `[i]` 인덱싱 불가
- **Async 버전은 Stage 2**: 동기 버전만 표준화 완료

## 실무 적용

가상화 리스트, 무한 스크롤, 스트리밍 API 처리 등 **전체를 버퍼링하지 않고 필요한 만큼만 소비**하는 시나리오에 적합.

```javascript
function* fetchPages() {
  let page = 1;
  while (true) yield fetch(`/api?page=${page++}`);
}

const first20 = fetchPages()
  .flatMap(res => res.items)
  .take(20)
  .toArray();
```

### 브라우저 지원

Chrome 122+, Firefox 131+, Safari 18+, Node 22+

## 출처 / 참고

- [Stop Turning Everything Into Arrays](https://allthingssmitty.com/2026/01/12/stop-turning-everything-into-arrays-and-do-less-work-instead/)
- [TC39 proposal-iterator-helpers](https://github.com/tc39/proposal-iterator-helpers)
- [ECMAScript 2025 Finalized](https://socket.dev/blog/ecmascript-2025-finalized)
- [[proxy|JavaScript Proxy]] — 관련 메타프로그래밍 개념

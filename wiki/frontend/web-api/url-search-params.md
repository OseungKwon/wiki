---
title: URLSearchParams
aliases: [url-search-params, 쿼리 파라미터]
tags:
  - javascript
  - web-api
  - url
created: 2026-03-18
updated: 2026-03-18
---

## 정의
URL의 쿼리 문자열(`?key=value&...`)을 파싱/조작하는 Web API 내장 객체.

## 상세 설명

### 생성 방법

```ts
// 문자열
const params = new URLSearchParams('?name=홍길동&age=30');

// URL 객체
const url = new URL('https://example.com?name=홍길동');
const params2 = url.searchParams;

// 객체
const params3 = new URLSearchParams({ name: '홍길동', age: '30' });
```

### 주요 메서드

| 메서드 | 설명 |
|--------|------|
| `get(key)` | 값 반환 (없으면 `null`) |
| `getAll(key)` | 같은 키의 모든 값 배열 반환 |
| `has(key)` | 키 존재 여부 |
| `set(key, value)` | 값 설정 (기존 덮어씀) |
| `append(key, value)` | 값 추가 (중복 허용) |
| `delete(key)` | 키 삭제 |
| `toString()` | 쿼리 문자열로 직렬화 |
| `entries()` / `keys()` / `values()` | 이터레이터 |

### 자동 인코딩/디코딩

`get()`은 **이미 디코딩된 값**을 반환한다. `decodeURIComponent()`를 추가로 호출하면 안 된다.

```ts
const params = new URLSearchParams('?nickname=%ED%8C%A8%EC%85%94');
params.get('nickname'); // "패셔" ← 자동 디코딩

// 위험 — 이미 디코딩된 값에 %가 포함되면 URIError 발생
decodeURIComponent(params.get('nickname'));

// 올바른 사용
params.get('nickname');
```

반대로 `toString()`은 자동으로 **인코딩**한다:

```ts
const params = new URLSearchParams({ nickname: '패셔' });
params.toString(); // "nickname=%ED%8C%A8%EC%85%94"
```

### 주의사항

- **값은 항상 문자열**: `get()`은 `string | null` 반환. 숫자 필요 시 직접 변환.
- **`+`를 공백으로 해석**: `?q=hello+world` → `get('q')` = `"hello world"`
- **같은 키 중복 가능**: `?a=1&a=2` → `get('a')` = `"1"`, `getAll('a')` = `["1", "2"]`

## 출처 / 참고
- [MDN - URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams)

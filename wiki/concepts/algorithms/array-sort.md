---
title: "JavaScript Array.sort() 심화"
aliases: [Array.sort, sort, TimSort]
tags:
  - javascript
  - sorting
  - array
  - timsort
created: 2026-04-07
updated: 2026-04-07
reviewed: false
---

## 정의
`Array.prototype.sort()`는 비교 함수 없이 호출 시 모든 요소를 `String()`으로 변환 후 유니코드 코드포인트 순으로 정렬한다. 숫자 배열 정렬에 `(a, b) => a - b` 비교 함수가 필수인 이유.

## 상세 설명

### 기본 정렬: 문자열 변환 후 유니코드 순

```typescript
[10, 9, 2, 1, 100].sort()  // → [1, 10, 100, 2, 9]
// 내부: "1"(49) < "10"(49,48) < "100" < "2"(50) < "9"(57)

[0, -1, -10, -2].sort()    // → [-1, -10, -2, 0] (음수도 깨짐)
[3, 1, 5, 9].sort()        // → [1, 3, 5, 9] (한 자리는 우연히 정상)
```

### 올바른 숫자 정렬

```typescript
arr.sort((a, b) => a - b)  // 오름차순 (음수→a앞, 0→유지, 양수→b앞)
arr.sort((a, b) => b - a)  // 내림차순
```

### 원본 변경 (mutation)

```typescript
const original = [3, 1, 2];
const result = original.sort((a, b) => a - b);
console.log(result === original);  // true — 같은 참조, 원본 바뀜
```

### toSorted (ES2023) — 원본 보존

```typescript
const sorted = arr.toSorted((a, b) => a - b);  // 새 배열 반환, 원본 유지
```

### 안정 정렬 (ES2019+)

같은 키 값의 원래 순서가 보장된다. ES2019 명세에서 표준화.

### 엔진별 내부 알고리즘

| 엔진 | 알고리즘 | 복잡도 |
|------|---------|--------|
| V8 (Chrome/Node) | **TimSort** (2019~) | O(n)~O(n log n), 공간 O(n) |
| SpiderMonkey (Firefox) | Merge Sort | O(n log n), 공간 O(n) |
| JavaScriptCore (Safari) | Merge Sort + InsertionSort | O(n log n), 공간 O(n) |

V8은 2019년 이전 QuickSort(불안정, 최악 O(n²))에서 TimSort로 전환.

### TimSort

Tim Peters가 2002년 Python용으로 설계한 Merge Sort + Insertion Sort 하이브리드.

1. 배열을 "run" 단위로 분할 (최소 32~64)
2. 각 run을 InsertionSort로 정렬
3. 이미 정렬된/역순인 구간은 그대로 활용
4. run들을 Merge Sort로 병합 (galloping mode로 최적화)

V8 동작: n ≤ 1 → no-op / n ≤ 64 → InsertionSort / n > 64 → TimSort

전환 이유: 안정성 보장(ES2019), 최악 O(n²) 제거, 부분 정렬 데이터에 유리.

## 출처 / 참고
- [[bucket-sort|Bucket Sort]] — 정렬 알고리즘 카테고리
- [[binary-search|Binary Search]] — 정렬된 배열 전제 알고리즘
- [[linked-list|연결 리스트]] — 같은 자료구조/알고리즘 카테고리

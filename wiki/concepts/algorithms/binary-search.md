---
title: Binary Search (이진 탐색)
aliases:
  - 이분 탐색
  - 이진 검색
tags:
  - algorithm
  - binary-search
  - search
created: 2026-04-01
updated: 2026-04-01
reviewed: false
---

## 정의

정렬된 데이터에서 탐색 범위를 매 단계 절반으로 줄여가며 목표값을 찾는 알고리즘. 시간 복잡도 O(log n).

## 상세 설명

### 전제 조건

데이터가 **정렬되어 있어야** 한다. 정렬되지 않으면 "왼쪽/오른쪽 중 어디를 버릴지" 판단할 수 없다.

### 패턴 1: 정확히 일치하는 값 찾기

배열에서 target과 일치하는 원소의 인덱스를 반환한다. 없으면 -1.

```typescript
function binarySearch(nums: number[], target: number): number {
  let lo = 0, hi = nums.length - 1;

  while (lo <= hi) {
    const mid = Math.floor((lo + hi) / 2);
    if (nums[mid] === target) return mid;
    if (nums[mid] < target) lo = mid + 1;
    else hi = mid - 1;
  }

  return -1;
}
```

### 패턴 2: Lower Bound (조건을 만족하는 첫 위치)

target 이상인 첫 번째 위치를 반환한다. 삽입 위치 찾기에 활용.

```typescript
function lowerBound(nums: number[], target: number): number {
  let lo = 0, hi = nums.length;  // 반열린 구간 [lo, hi)

  while (lo < hi) {
    const mid = Math.floor((lo + hi) / 2);
    if (nums[mid] < target) lo = mid + 1;
    else hi = mid;  // mid를 후보에 포함
  }

  return lo;
}
```

### 두 패턴의 차이

| | 정확히 찾기 | Lower Bound |
|---|---|---|
| 루프 조건 | `lo <= hi` | `lo < hi` |
| hi 초기값 | `length - 1` | `length` |
| 찾았을 때 | 즉시 `return mid` | `hi = mid` (계속 좁힘) |
| 반환값 | 인덱스 또는 -1 | 조건 만족하는 첫 위치 |

### Lower Bound 핵심 포인트

- `hi = nums.length`인 이유: 삽입 위치가 배열 끝 다음일 수 있으므로 반열린 구간 사용
- `hi = mid`인 이유: `nums[mid] >= target`이면 mid가 답일 수 있으므로 후보에 포함
- `lo = mid + 1`인 이유: `nums[mid] < target`이면 mid와 왼쪽은 답이 될 수 없음

### 변형: Upper Bound

target **초과**인 첫 번째 위치를 찾으려면 조건만 변경:

```typescript
// Lower Bound: nums[mid] < target  → lo = mid + 1  (첫 번째 ≥ target)
// Upper Bound: nums[mid] <= target → lo = mid + 1  (첫 번째 > target)
```

## 실무 적용

- **Search Insert Position** (LeetCode 35): Lower Bound의 기본형. 정렬된 배열에서 target의 삽입 위치를 O(log n)으로 찾는 문제.
- **Parametric Search**: 답의 범위를 이진 탐색으로 좁히는 패턴. "최소값의 최대화", "최대값의 최소화" 유형에서 활용.

## 출처 / 참고

- LeetCode 35. Search Insert Position
- LeetCode 704. Binary Search
- [[bucket-sort]] — 같은 알고리즘 카테고리

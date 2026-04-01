---
date: 2026-04-01
category: binary-search
difficulty: easy
source: LeetCode
result: pass
hints: 0
retries: 2
expected_time: 10~15min
---

## 문제
- **링크**: https://leetcode.com/problems/search-insert-position/
- **설명**: 정렬된 배열에서 target의 인덱스 또는 삽입 위치를 O(log n)으로 반환
- **적정 풀이시간**: 10~15분

## 풀이
### 접근 방식
`findIndex`를 활용한 선형 탐색. 먼저 target과 일치하는 원소를 찾고, 없으면 target보다 큰 첫 번째 원소의 위치를 반환.

### 코드
```typescript
function searchInsert(nums: number[], target: number): number {
    let index = nums.findIndex(num => num === target);
    if (index !== -1) {
        return index;
    }
    index = nums.findIndex(num => num > target);
    if (index === -1) {
        return nums.length;
    }
    return index;
}
```

## 코드 리뷰
### 정확성
- 실제 제출: 통과
- 잠재 취약점: 특이사항 없음

### 시간/공간 복잡도
- 시간: O(n) / 공간: O(1)
- 문제에서 O(log n)을 요구하므로 이진 탐색으로 최적화 가능

### 개선점
1. O(log n) 이진 탐색으로 변경 — 문제의 시간 복잡도 요구사항 충족
2. findIndex 두 번 호출 대신 한 번의 탐색으로 통합

### 개선된 코드
```typescript
// Lower Bound 이진 탐색으로 변경. O(n) → O(log n)
function searchInsert(nums: number[], target: number): number {
  let lo = 0;
  let hi = nums.length;

  while (lo < hi) {
    const mid = Math.floor((lo + hi) / 2);
    if (nums[mid] < target) {
      lo = mid + 1;
    } else {
      hi = mid;
    }
  }

  return lo;
}
```

### 핵심 takeaway
Search Insert Position은 Lower Bound 이진 탐색의 기본형. `lo < hi` + `hi = mid` 패턴을 익혀두면 변형 문제에 바로 적용 가능.

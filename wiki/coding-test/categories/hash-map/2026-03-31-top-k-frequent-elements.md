---
date: 2026-03-31
category: hash-map
difficulty: medium
source: LeetCode
result: pass
hints: 0
retries: 1
expected_time: 20~35min
retry_from_keep: false
---

## 문제
- **링크**: https://leetcode.com/problems/top-k-frequent-elements/
- **설명**: 정수 배열에서 가장 자주 등장하는 k개의 요소를 반환

## 풀이
### 접근 방식
Map으로 빈도 집계 후 entries를 빈도 내림차순 정렬하여 상위 k개 추출

### 코드
```typescript
function answer(nums, k){
    const map = new Map();
    nums.forEach(num => {
        if(map.has(num)) {
            map.set(num, map.get(num) + 1)
        } else {
            map.set(num, 1)
        }
    })

    return [...map.entries()].sort((a, b) => b[1] - a[1]).map(data => data[0]).slice(0, k)
}
```

## 코드 리뷰
### 정확성
- 모든 테스트케이스 통과. 로직 정확

### 시간/공간 복잡도
- 시간: O(n log n) — sort() 사용. 문제 요구사항(O(n log n) 미만)을 엄밀히 만족하지 못함
- 공간: O(n)

### 개선점
1. `map.set(num, (map.get(num) ?? 0) + 1)` — if/else 분기 제거
2. Bucket Sort로 O(n) 달성 가능: index=빈도, value=숫자 배열

### 개선된 코드
```typescript
// 개선: if/else→?? 0, 정렬→Bucket Sort로 O(n log n)→O(n), TypeScript 타입 추가
function topKFrequent(nums: number[], k: number): number[] {
  const freqMap = new Map<number, number>();
  for (const num of nums) {
    freqMap.set(num, (freqMap.get(num) ?? 0) + 1);
  }
  const buckets: number[][] = Array.from({ length: nums.length + 1 }, () => []);
  for (const [num, freq] of freqMap) {
    buckets[freq].push(num);
  }
  const result: number[] = [];
  for (let i = buckets.length - 1; i >= 0 && result.length < k; i--) {
    result.push(...buckets[i]);
  }
  return result.slice(0, k);
}
```

### 핵심 takeaway
정렬 기반 Top-K는 O(n log n). Bucket Sort로 O(n) 달성 가능 — 빈도의 최대값이 n으로 제한되는 점을 활용

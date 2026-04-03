---
title: Bucket Sort
aliases: [버킷 정렬]
tags:
  - algorithm
  - sorting
  - bucket-sort
  - hash-map
created: 2026-03-31
updated: 2026-03-31
reviewed: false
---

## 정의
값의 범위가 제한적일 때 배열의 인덱스 자체를 정렬 키로 사용하여 O(n)에 정렬하는 비교 기반이 아닌 정렬 알고리즘.

## 상세 설명

### 동작 원리
1. 값의 범위만큼 빈 배열(bucket)을 생성
2. 각 요소를 해당하는 인덱스의 버킷에 삽입
3. 버킷을 순서대로 순회하여 결과 수집

### 왜 O(n)인가
- 버킷에 넣기: O(n)
- 버킷 순회: O(범위) (범위가 n에 비례하면 O(n))
- 비교 정렬(sort)을 사용하지 않으므로 O(n log n) 한계를 돌파

### 사용 조건
값의 범위가 유한하고 알려져 있어야 한다. 범위가 n보다 훨씬 크면 공간 낭비가 발생한다.

## 실무 적용

### Top K Frequent Elements (LeetCode 347)
빈도 기반 Top-K 문제에서 빈도를 인덱스로 사용하는 변형.

```typescript
// Step 1: 빈도 집계
const freqMap = new Map<number, number>();
for (const num of nums) {
  freqMap.set(num, (freqMap.get(num) ?? 0) + 1);
}

// Step 2: 빈도를 인덱스로 하는 버킷 생성
// index = 빈도, value = 해당 빈도의 숫자 배열
const buckets: number[][] = Array.from({ length: nums.length + 1 }, () => []);
for (const [num, freq] of freqMap) {
  buckets[freq].push(num);
}

// Step 3: 뒤에서부터 순회하며 k개 수집
const result: number[] = [];
for (let i = buckets.length - 1; i >= 0 && result.length < k; i--) {
  result.push(...buckets[i]);
}
return result.slice(0, k);
```

- 빈도의 최대값이 `nums.length`로 제한되므로 버킷 크기가 유한
- 동일 빈도 원소는 같은 버킷에 모두 저장됨
- 시간: O(n) / 공간: O(n)

### 관련 알고리즘
- [[counting-sort]]: 값 자체를 인덱스로 사용 (bucket sort의 특수 케이스)
- Radix Sort: 자릿수별로 bucket sort 반복

## 출처 / 참고
- LeetCode 347. Top K Frequent Elements

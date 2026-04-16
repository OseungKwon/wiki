---
date: 2026-04-16
category: heap
difficulty: medium
source: LeetCode
result: pass
hints: 0
retries: 2
expected_time: 20~35min
---

## 문제
- **링크**: https://leetcode.com/problems/kth-largest-element-in-an-array/
- **설명**: 정수 배열 nums와 k가 주어질 때, k번째로 큰 수를 반환하라.
- **적정 풀이시간**: 20~35min

## 풀이
### 접근 방식
Max-heap을 직접 구현. 모든 원소를 push한 뒤 k번 pop → k번째로 큰 수.
(첫 시도는 min-heap으로 잘못 구현 → k번째 작은 수가 나옴. 비교 방향 수정 후 통과.)

### 코드
```typescript
function answer(nums: number[], k: number): number {
    const heap: number[] = [];

    function hpush(val: number) {
        heap.push(val);
        let curr = heap.length - 1;
        while (curr > 0) {
            const parent = (curr - 1) >> 1;
            if (heap[parent] < heap[curr]) {
                [heap[parent], heap[curr]] = [heap[curr], heap[parent]];
                curr = parent;
            } else break;
        }
    }

    function hpop(): number {
        if (heap.length === 1) return heap.pop()!;
        const top = heap[0];
        heap[0] = heap.pop()!;
        let curr = 0;
        while (true) {
            let largest = curr;
            const left = 2 * curr + 1, right = 2 * curr + 2;
            if (left < heap.length && heap[left] > heap[largest]) largest = left;
            if (right < heap.length && heap[right] > heap[largest]) largest = right;
            if (largest !== curr) {
                [heap[largest], heap[curr]] = [heap[curr], heap[largest]];
                curr = largest;
            } else break;
        }
        return top;
    }

    for (const num of nums) hpush(num);
    let result = 0;
    for (let i = 0; i < k; i++) result = hpop();
    return result;
}
```

## 코드 리뷰
### 정확성
- 실제 제출: 통과
- 잠재 취약점: 특이사항 없음

### 시간/공간 복잡도
- 시간: O(n log n) — 전체 push + k번 pop
- 공간: O(n)
- 개선 가능: size-k min-heap으로 O(n log k)

### 개선된 코드
```typescript
// size-k min-heap: O(n log k)
for (const num of nums) {
    hpush(num);
    if (heap.length > k) hpop();
}
return heap[0]; // k개 중 최솟값 = 전체 k번째로 큰 값
```

### 핵심 takeaway
min/max heap은 부모↔자식 비교 방향(`>`↔`<`) 하나 차이. size-k min-heap이 top-K 문제의 표준 패턴.

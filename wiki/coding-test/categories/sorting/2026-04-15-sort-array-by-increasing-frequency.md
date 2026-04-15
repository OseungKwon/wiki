---
date: 2026-04-15
category: sorting
difficulty: easy
source: LeetCode
result: pass
hints: 0
retries: 1
expected_time: 10~15min
---

## 문제
- **링크**: https://leetcode.com/problems/sort-array-by-increasing-frequency/
- **설명**: 정수 배열을 빈도 오름차순으로 정렬. 빈도 같으면 값 내림차순.
- **적정 풀이시간**: 10~15분

## 풀이
### 접근 방식
Map으로 빈도 카운팅 후, Map.entries()를 빈도 1순위 / 값 내림차순 2순위로 정렬. flatMap으로 펼쳐 반환.

### 코드
```typescript
function answer(nums: number[]): number[] {
    const map = new Map<number, number>();
    nums.forEach(n => {
        map.set(n, (map.get(n) ?? 0) + 1);
    });
    return Array.from(map.entries())
        .sort((a, b) => {
            if (a[1] < b[1]) return -1;
            if (a[1] > b[1]) return 1;
            if (b[0] > a[0]) return 1;
            if (b[0] < a[0]) return -1;
            return 0;
        })
        .flatMap(([value, count]) => Array.from({ length: count }, () => value));
}
```

## 코드 리뷰
### 정확성
LeetCode 제출 통과. 잠재 취약점 없음 (제약 조건 내 안전).

### 시간/공간 복잡도
- 시간: O(n log n) — 카운팅 O(n) + 정렬 O(k log k)
- 공간: O(n) — Map + 결과 배열

### 개선점
1. 비교 함수를 삼항 연산자로 단순화 (if 4개 → 1줄)
2. 구조분해할당으로 `a[0]`, `a[1]` 제거
3. `Array.from({ length: count }, () => value)` → `new Array(count).fill(value)`

### 개선된 코드
```typescript
function answer(nums: number[]): number[] {
    const freq = new Map<number, number>();
    for (const n of nums) {
        freq.set(n, (freq.get(n) ?? 0) + 1);
    }
    return [...freq.entries()]
        .sort(([va, fa], [vb, fb]) => fa !== fb ? fa - fb : vb - va)
        .flatMap(([v, c]) => new Array(c).fill(v));
}
```

### 핵심 takeaway
다중 기준 정렬은 `if (기준 다르면) return 차이; return 다음기준 차이;` early return 패턴으로 충분하다.

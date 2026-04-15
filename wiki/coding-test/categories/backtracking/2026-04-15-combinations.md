---
date: 2026-04-15
category: backtracking
difficulty: medium
source: LeetCode
result: pass
hints: 0
retries: 1
expected_time: 20~35min
---

## 문제
- **링크**: https://leetcode.com/problems/combinations/
- **설명**: 1부터 n까지의 수 중 정확히 k개를 선택한 모든 조합을 반환하라.
- **적정 풀이시간**: 20~35min

## 풀이
### 접근 방식
Backtracking. `current.length === k`를 종료 조건으로, `i + 1`로 순방향 탐색 보장.

### 코드
```typescript
function answer(n: number, k: number): number[][] {
    const result: number[][] = [];
    function recursive(index: number, current: number[]) {
        if (current.length === k) {
            result.push([...current]);
            return;
        }
        for (let i = index; i < n; i++) {
            current.push(i + 1);
            recursive(i + 1, current);
            current.pop();
        }
    }
    recursive(0, []);
    return result;
}
```

## 코드 리뷰
### 정확성
- 실제 제출: 통과
- 잠재 취약점: 특이사항 없음

### 시간/공간 복잡도
- 시간: O(k × C(n,k))
- 공간: O(k)

### 개선점
가지치기 추가: `if (n - i < k - current.length) break;` → 남은 숫자가 부족하면 조기 종료

### 개선된 코드
```typescript
function combine(n: number, k: number): number[][] {
    const result: number[][] = [];
    function backtrack(index: number, current: number[]) {
        if (current.length === k) {
            result.push([...current]);
            return;
        }
        for (let i = index; i < n; i++) {
            if (n - i < k - current.length) break; // 가지치기
            current.push(i + 1);
            backtrack(i + 1, current);
            current.pop();
        }
    }
    backtrack(0, []);
    return result;
}
```

### 핵심 takeaway
`i + 1`로 순방향 탐색 보장 + `current.length === k` 종료 조건 = Backtracking 조합 생성 기본 구조

---
date: 2026-03-31
category: dynamic-programming
difficulty: easy
source: LeetCode
result: pass
hints: 0
retries: 1
expected_time: 10~15min
retry_from_keep: false
---

## 문제
- **링크**: https://leetcode.com/problems/climbing-stairs/
- **설명**: n개 계단을 1칸 또는 2칸씩 오르는 방법의 수

## 풀이
### 접근 방식
Bottom-up DP. dp[i] = dp[i-1] + dp[i-2] 점화식 사용

### 코드
```typescript
function answer(n) {
    const dp = new Array(n + 1).fill(0);
    dp[1] = 1;
    dp[2] = 2;
    for (let i = 3; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }
    return dp[n];
}
```

## 코드 리뷰
### 정확성
- 모든 테스트케이스 통과

### 시간/공간 복잡도
- 시간: O(n) / 공간: O(n)
- 공간 최적화 가능: 변수 2개로 O(1)

### 개선점
- 이전 두 값만 필요하므로 배열 대신 변수 2개 사용 가능

### 개선된 코드
```typescript
// 개선: 배열→변수 2개로 공간 O(n)→O(1), TypeScript 타입 추가
function climbStairs(n: number): number {
  if (n <= 2) return n;
  let prev = 1, curr = 2;
  for (let i = 3; i <= n; i++) {
    [prev, curr] = [curr, prev + curr];
  }
  return curr;
}
```

### 핵심 takeaway
DP 핵심은 점화식 도출 + base case 설정. 공간 최적화는 "이전 몇 개만 필요한가"를 파악하는 것

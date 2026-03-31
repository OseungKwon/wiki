---
category: dynamic-programming
mastery: 1
total_solved: 1
pass_rate: 100%
last_solved: 2026-03-31
---

## 서브패턴 현황

| 서브패턴 | 수준 | 경험 | pass_rate | 대표 문제 |
|----------|------|------|-----------|----------|
| 1d-linear | F | O | 100% | Climbing Stairs |
| 2d-grid | F | X | — | Unique Paths |
| knapsack | F | X | — | 0/1 Knapsack |
| LIS | F | X | — | Longest Increasing Subsequence |
| interval-dp | I | X | — | Burst Balloons |
| state-machine | I | X | — | Best Time to Buy and Sell Stock |
| bitmask-dp | A | X | — | Travelling Salesman |

## 핵심 패턴

### 1. Bottom-up 1D Linear DP (피보나치 변형)
- 언제: 현재 상태가 이전 1~2개 상태에만 의존할 때
- 핵심: 점화식 도출 → base case 설정 → 순회
- 공간 최적화: 이전 값만 필요하면 배열 대신 변수 사용 → O(1)
- 템플릿:
```typescript
let prev = 1, curr = 2;
for (let i = 3; i <= n; i++) {
  [prev, curr] = [curr, prev + curr];
}
```

## 자주 하는 실수

> 풀이가 쌓이면 자동으로 추가됩니다.

## 풀이 목록

| 날짜 | 문제 | 서브패턴 | 난이도 | 결과 | 소요시간 | 링크 |
|------|------|----------|--------|------|----------|------|
| 2026-03-31 | Climbing Stairs | 1d-linear | easy | pass | - | [[2026-03-31-climbing-stairs]] |

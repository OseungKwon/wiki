---
category: two-pointer
mastery: 2.5
total_solved: 1
pass_rate: 100%
weighted_pass_rate: 100%
last_solved: 2026-03-31
---

## 서브패턴 현황

| 서브패턴 | 수준 | 경험 | pass_rate | 대표 문제 |
|----------|------|------|-----------|----------|
| 양끝 포인터 | F | O | 100% | Two Sum II |
| 정렬 후 탐색 | F | X | 0% | — |
| 3Sum/4Sum | I | X | 0% | — |
| Container With Most Water | I | X | 0% | — |
| Trapping Rain Water | A | X | 0% | — |
| 다중 포인터 | A | X | 0% | — |

## 핵심 패턴

- **양끝 포인터**: 정렬된 배열에서 합이 크면 right--, 작으면 left++. O(n) 탐색.

## 자주 하는 실수

- 1-based vs 0-based 인덱스 혼동 주의

## 풀이 목록

| 날짜 | 문제 | 서브패턴 | 난이도 | 결과 | 힌트 | 재시도 | 적정시간 | 링크 |
|------|------|----------|--------|------|------|--------|----------|------|
| 2026-03-31 | Two Sum II | 양끝 포인터 | medium | pass | 0 | 2 | 20~35min | [[2026-03-31-two-sum-ii]] |

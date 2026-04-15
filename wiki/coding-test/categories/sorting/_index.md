---
category: sorting
mastery: 1.0
total_solved: 1
pass_rate: 100%
weighted_pass_rate: 100%
last_solved: 2026-04-15
---

## 서브패턴 현황

| 서브패턴 | 수준 | 경험 | pass_rate | 대표 문제 |
|----------|------|------|-----------|----------|
| 기본 정렬, 커스텀 비교 | F | O | 100% | Sort Array by Increasing Frequency |
| Merge Sort 응용, Quick Select | I | X | 0% | — |
| Counting/Radix Sort, 외부 정렬 개념 | A | X | 0% | — |

## 핵심 패턴

- **Multi-key Comparator**: `if (기준A 다르면) return 차이; return 기준B 차이;` — early return으로 우선순위 분기, else 불필요
- 구조분해할당으로 `a[0]`, `a[1]` 대신 `[value, freq]`로 의미를 드러내기

## 자주 하는 실수

> 리뷰에서 반복되는 실수가 발견되면 여기에 추가됩니다.

## 풀이 목록

| 날짜 | 문제 | 서브패턴 | 난이도 | 결과 | 힌트 | 재시도 | 적정시간 | 링크 |
|------|------|----------|--------|------|------|--------|----------|------|
| 2026-04-15 | Sort Array by Increasing Frequency | 커스텀 비교 | easy | pass | 0 | 1 | 10~15min | [[categories/sorting/2026-04-15-sort-array-by-increasing-frequency]] |

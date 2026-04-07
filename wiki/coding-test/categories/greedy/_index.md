---
category: greedy
mastery: 1.5
total_solved: 1
pass_rate: 100%
weighted_pass_rate: 100%
last_solved: 2026-04-07
---

## 서브패턴 현황

| 서브패턴 | 수준 | 경험 | pass_rate | 대표 문제 |
|----------|------|------|-----------|----------|
| 활동 선택 | F | O | 100% | Assign Cookies |
| 거스름돈 | F | X | 0% | — |
| Interval Scheduling | I | X | 0% | — |
| Huffman 개념 | I | X | 0% | — |
| Exchange Argument 증명 | A | X | 0% | — |
| 매트로이드 | A | X | 0% | — |

## 핵심 패턴

- **정렬 후 탐욕적 매칭**: 양쪽 배열 정렬 → 큰 것부터(또는 작은 것부터) 매칭. 최적 부분 구조 활용.

## 자주 하는 실수

- JS `.sort()` 사전순 정렬 함정 — 숫자 정렬 시 반드시 `(a, b) => a - b`

## 풀이 목록

| 날짜 | 문제 | 서브패턴 | 난이도 | 결과 | 힌트 | 재시도 | 적정시간 | 링크 |
|------|------|----------|--------|------|------|--------|----------|------|
| 2026-04-07 | Assign Cookies | 활동 선택 | easy | pass | 0 | 2 | 10~15min | [[categories/greedy/2026-04-07-assign-cookies]] |

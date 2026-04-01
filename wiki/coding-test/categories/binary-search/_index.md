---
category: binary-search
mastery: 1
total_solved: 1
pass_rate: 100%
weighted_pass_rate: 100%
last_solved: 2026-04-01
---

## 서브패턴 현황

| 서브패턴 | 수준 | 경험 | pass_rate | 대표 문제 |
|----------|------|------|-----------|----------|
| 기본 이분 탐색 | F | O | 100% | Search Insert Position |
| Lower/Upper Bound | F | O | 100% | Search Insert Position |
| 답 이분 탐색 (Parametric Search) | I | X | 0% | — |
| 2D 이분 탐색 | A | X | 0% | — |
| 최적화 문제 | A | X | 0% | — |

## 핵심 패턴

- **Lower Bound 패턴**: `lo < hi` 루프 + `nums[mid] < target → lo = mid+1` / `else → hi = mid`. 종료 시 `lo`가 target 이상인 첫 위치.

## 자주 하는 실수

- target이 모든 원소보다 작을 때(맨 앞 삽입) 처리 누락

## 풀이 목록

| 날짜 | 문제 | 서브패턴 | 난이도 | 결과 | 힌트 | 재시도 | 적정시간 | 링크 |
|------|------|----------|--------|------|------|--------|----------|------|
| 2026-04-01 | Search Insert Position | 기본 이분 탐색 | easy | pass | 0 | 2 | 10~15min | [[categories/binary-search/2026-04-01-search-insert-position]] |

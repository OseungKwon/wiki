---
category: hash-map
mastery: 1
total_solved: 1
pass_rate: 100%
last_solved: 2026-03-31
---

## 서브패턴 현황

| 서브패턴 | 수준 | 경험 | pass_rate | 대표 문제 |
|----------|------|------|-----------|----------|
| frequency-count | F | O | 100% | Top K Frequent Elements |
| grouping | F | X | — | Group Anagrams |
| two-sum-pattern | F | X | — | Two Sum |
| caching | F | X | — | LRU Cache |
| set-operations | F | X | — | Intersection of Two Arrays |

## 핵심 패턴

### 1. Frequency Count + Bucket Sort
- 언제: 빈도 기반 Top-K 문제
- 핵심: Map으로 빈도 집계 후, 배열 인덱스를 빈도로 사용하는 Bucket Sort로 O(n) 달성
- 포인트: 빈도 최대값이 배열 길이로 제한되므로 bucket 크기가 유한

## 자주 하는 실수

- [ ] `map.has()` + `map.get()` 분기 대신 `(map.get(key) ?? 0) + 1` 사용할 것
- [ ] Top-K에 sort() 사용 시 O(n log n) — Bucket Sort나 Heap 고려

## 풀이 목록

| 날짜 | 문제 | 서브패턴 | 난이도 | 결과 | 소요시간 | 링크 |
|------|------|----------|--------|------|----------|------|
| 2026-03-31 | Top K Frequent Elements | frequency-count | medium | pass | - | [[2026-03-31-top-k-frequent-elements]] |

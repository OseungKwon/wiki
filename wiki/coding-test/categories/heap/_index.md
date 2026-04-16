---
category: heap
mastery: 2.5
total_solved: 1
pass_rate: 100%
weighted_pass_rate: 100%
last_solved: 2026-04-16
---

## 서브패턴 현황

| 서브패턴 | 수준 | 경험 | pass_rate | 대표 문제 |
|----------|------|------|-----------|----------|
| 기본 힙 연산, Top-K | F | O | 100% | Kth Largest Element |
| 이중 힙 (중앙값), Merge K Sorted | I | X | 0% | — |
| Lazy Deletion, 커스텀 힙 | A | X | 0% | — |

## 핵심 패턴

- **min/max heap 전환**: 부모↔자식 비교 방향(`>`↔`<`) 하나만 다름
- **Top-K 표준 패턴**: size-k min-heap 유지 → top이 k번째로 큰 값 (O(n log k))
- hpush: push → 위로 버블업 / hpop: 루트 제거 → 마지막 원소를 루트로 → 아래로 버블다운

## 자주 하는 실수

- min/max heap 비교 방향 혼동: min-heap에서 k번 pop → k번째 **작은** 수가 나옴 (largest 아님)

## 풀이 목록

| 날짜 | 문제 | 서브패턴 | 난이도 | 결과 | 힌트 | 재시도 | 적정시간 | 링크 |
|------|------|----------|--------|------|------|--------|----------|------|
| 2026-04-16 | Kth Largest Element in an Array | 기본 힙 연산, Top-K | medium | pass | 0 | 2 | 20~35min | [[categories/heap/2026-04-16-kth-largest-element]] |

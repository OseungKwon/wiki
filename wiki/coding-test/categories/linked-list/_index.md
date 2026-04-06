---
category: linked-list
mastery: 1
total_solved: 2
pass_rate: 0%
weighted_pass_rate: 0%
last_solved: 2026-04-06
---

## 서브패턴 현황

| 서브패턴 | 수준 | 경험 | pass_rate | 대표 문제 |
|----------|------|------|-----------|----------|
| 순회/삽입/삭제 | F | O | 0% | Remove Duplicates from Sorted List |
| 역순 | F | O | 0% | Reverse Linked List |
| Fast-Slow 포인터 | I | X | 0% | — |
| 병합 | I | X | 0% | — |
| LRU Cache | A | X | 0% | — |
| Skip List 개념 | A | X | 0% | — |

## 핵심 패턴

- **포인터 3개 역순 패턴**: prev/curr/next로 in-place 뒤집기. `curr.next = prev`가 핵심, 뒤집기 전에 `next = curr.next`로 임시 저장 필수.
- **중복 건너뛰기 패턴**: 정렬된 리스트에서 `curr.val === curr.next.val`이면 `curr.next = curr.next.next`. 같을 때 curr를 이동하지 않아야 연속 중복 처리 가능.

## 자주 하는 실수

- 배열로 값을 꺼내서 처리하려는 접근 — ListNode 포인터 조작이 핵심임을 인식해야 함
- `node.val`로 조건 체크 시 val=0이면 순회 종료되는 버그

## 풀이 목록

| 날짜 | 문제 | 서브패턴 | 난이도 | 결과 | 힌트 | 재시도 | 적정시간 | 링크 |
|------|------|----------|--------|------|------|--------|----------|------|
| 2026-04-06 | Reverse Linked List | 역순 | easy | fail | 3 | 0 | 10~15min | [[categories/linked-list/2026-04-06-reverse-linked-list]] |
| 2026-04-06 | Remove Duplicates from Sorted List | 순회/삽입/삭제 | easy | fail | 3 | 0 | 10~15min | [[categories/linked-list/2026-04-06-remove-duplicates-from-sorted-list]] |

---
category: stack-queue
mastery: 5
total_solved: 1
pass_rate: 100%
weighted_pass_rate: 100%
last_solved: 2026-03-31
---

## 서브패턴 현황

| 서브패턴 | 수준 | 경험 | pass_rate | 대표 문제 |
|----------|------|------|-----------|----------|
| 괄호 매칭 | F | O | 100% | Valid Parentheses |
| 역순 처리 | F | X | 0% | — |
| Monotonic Stack | I | X | 0% | — |
| Min Stack | I | X | 0% | — |
| Largest Rectangle in Histogram | A | X | 0% | — |
| 슬라이딩 윈도우 최대값 | A | X | 0% | — |

## 핵심 패턴

- **괄호 매칭**: 여는 괄호 push, 닫는 괄호 만나면 stack top과 비교. Map으로 닫는→여는 매핑하면 깔끔.

## 자주 하는 실수

- forEach 콜백 안의 return은 외부 함수를 종료하지 않음 → 조기 종료가 필요하면 for 루프 사용

## 풀이 목록

| 날짜 | 문제 | 서브패턴 | 난이도 | 결과 | 힌트 | 재시도 | 적정시간 | 링크 |
|------|------|----------|--------|------|------|--------|----------|------|
| 2026-03-31 | Valid Parentheses | 괄호 매칭 | easy | pass | 0 | 2 | 10~15min | [[2026-03-31-valid-parentheses]] |

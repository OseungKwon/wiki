---
category: backtracking
mastery: "2.5"
total_solved: "2"
pass_rate: 50%
weighted_pass_rate: 50%
last_solved: 2026-04-15
---

## 서브패턴 현황

| 서브패턴 | 수준 | 경험 | pass_rate | 대표 문제 |
|----------|------|------|-----------|----------|
| 순열/조합 생성, N-Queen | F | O | 0% | Subsets |
| 제약 전파, 가지치기 최적화 | I | X | 0% | — |
| 복합 조건 탐색, 게임 트리 | A | X | 0% | — |

## 핵심 패턴

- **Backtracking 기본 구조**: `result.push([...current])` → for loop → `push` → `backtrack(i+1)` → `pop`
- for 루프 시작을 `index`로 고정해 중복 부분집합 방지

## 자주 하는 실수

- result.push 시점을 반복문 안에 넣으면 빈 집합이 누락됨
- current를 복사(`[...current]`)하지 않으면 참조 문제로 결과가 모두 같아짐

## 풀이 목록

| 날짜 | 문제 | 서브패턴 | 난이도 | 결과 | 힌트 | 재시도 | 적정시간 | 링크 |
|------|------|----------|--------|------|------|--------|----------|------|
| 2026-04-15 | Subsets | 순열/조합 생성 | medium | fail | 3 | 0 | 20~35min | [[categories/backtracking/2026-04-15-subsets]] |
| 2026-04-15 | Combinations | 순열/조합 생성 | medium | pass | 0 | 1 | 20~35min | [[categories/backtracking/2026-04-15-combinations]] |

---
category: graph
mastery: 1
total_solved: 1
pass_rate: 100%
last_solved: 2026-03-31
---

## 서브패턴 현황

| 서브패턴 | 수준 | 경험 | pass_rate | 대표 문제 |
|----------|------|------|-----------|----------|
| dfs | F | O | 100% | Number of Islands |
| bfs | F | X | — | Rotting Oranges |
| shortest-path | F | X | — | Network Delay Time |
| topological-sort | I | X | — | Course Schedule |
| union-find | I | X | — | Number of Connected Components |
| mst | A | X | — | Min Cost to Connect All Points |

## 핵심 패턴

### 1. Grid DFS (Flood Fill)
- 언제: 2D 그리드에서 연결 영역 탐색/카운트
- 핵심: 방문 마킹 + 4방향 재귀
- 템플릿:
```typescript
const dirs = [[1,0],[0,1],[-1,0],[0,-1]];
function dfs(r: number, c: number) {
  for (const [dr, dc] of dirs) {
    const nr = r + dr, nc = c + dc;
    if (nr >= 0 && nc >= 0 && nr < rows && nc < cols && grid[nr][nc] === '1') {
      grid[nr][nc] = '2'; // 방문 마킹
      dfs(nr, nc);
    }
  }
}
```
- 원본 배열 수정으로 별도 visited 배열 불필요

## 자주 하는 실수

- [ ] dfs 파라미터에 불필요한 기본값 할당 (`grid = []`)
- [ ] 방향 배열을 재귀 함수 내부에서 매번 생성 → 외부로 분리

## 풀이 목록

| 날짜 | 문제 | 서브패턴 | 난이도 | 결과 | 소요시간 | 링크 |
|------|------|----------|--------|------|----------|------|
| 2026-03-31 | Number of Islands | dfs | medium | pass | - | [[2026-03-31-number-of-islands]] |

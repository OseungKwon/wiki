---
date: 2026-03-31
category: graph
difficulty: medium
source: LeetCode
result: pass
hints: 0
retries: 1
expected_time: 20~35min
---

## 문제
- **링크**: https://leetcode.com/problems/number-of-islands/
- **설명**: 2D 그리드에서 '1'(땅)로 이루어진 섬의 개수를 반환

## 풀이
### 접근 방식
그리드를 순회하며 '1'을 발견하면 카운트 증가 + DFS로 연결된 모든 땅을 '2'로 마킹

### 코드
```typescript
function answer(grid) {
    function dfs(pointer, grid = []) {
        const move = [[1,0],[0,1],[-1,0],[0,-1]];
        for (let i = 0; i < move.length; i++) {
            if (
                pointer[0] + move[i][0] >= 0 &&
                pointer[1] + move[i][1] >= 0 &&
                pointer[0] + move[i][0] < grid.length &&
                pointer[1] + move[i][1] < grid[0].length
            ) {
                const next = [pointer[0] + move[i][0], pointer[1] + move[i][1]];
                if (grid[next[0]][next[1]] === '1') {
                    grid[next[0]][next[1]] = '2';
                    dfs(next, grid);
                }
            }
        }
    }
    let cnt = 0;
    for (let i = 0; i < grid.length; i++) {
        for (let j = 0; j < grid[0].length; j++) {
            if (grid[i][j] === '1') {
                cnt++;
                dfs([i, j], grid);
            }
        }
    }
    return cnt;
}
```

## 코드 리뷰
### 정확성
- 모든 테스트케이스 통과

### 시간/공간 복잡도
- 시간: O(m*n) / 공간: O(m*n) 재귀 스택

### 개선점
- dfs 파라미터 `grid = []` 기본값 불필요
- 방향 배열을 함수 밖으로 분리
- 구조 분해 할당으로 가독성 향상

### 개선된 코드
```typescript
// 개선: grid 기본값 제거, 방향 배열 외부 분리, 구조 분해 할당, 시작점 마킹 추가
const DIRECTIONS = [[1,0],[0,1],[-1,0],[0,-1]] as const;

function numIslands(grid: string[][]): number {
  function dfs(row: number, col: number): void {
    for (const [dr, dc] of DIRECTIONS) {
      const nr = row + dr;
      const nc = col + dc;
      if (nr >= 0 && nc >= 0 && nr < grid.length && nc < grid[0].length && grid[nr][nc] === '1') {
        grid[nr][nc] = '2';
        dfs(nr, nc);
      }
    }
  }

  let cnt = 0;
  for (let i = 0; i < grid.length; i++) {
    for (let j = 0; j < grid[0].length; j++) {
      if (grid[i][j] === '1') {
        cnt++;
        grid[i][j] = '2';
        dfs(i, j);
      }
    }
  }
  return cnt;
}
```

### 핵심 takeaway
Grid DFS는 "방문 마킹 + 4방향 재귀"가 기본 골격. 원본 배열 수정으로 visited 배열 불필요

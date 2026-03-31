---
date: 2026-03-31
category: tree
difficulty: easy
source: LeetCode
result: pass (힌트1)
time: -
---

## 문제
- **링크**: https://leetcode.com/problems/maximum-depth-of-binary-tree/
- **설명**: 이진 트리의 최대 깊이를 반환

## 풀이
### 접근 방식
DFS 재귀로 왼쪽/오른쪽 자식의 깊이를 구한 뒤 1 + max(left, right) 반환

### 코드
```typescript
function answer(root) {
    function traverse(node) {
        if (node === null) {
            return 0;
        }
        const leftDepth = traverse(node.left);
        const rightDepth = traverse(node.right);
        return 1 + Math.max(leftDepth, rightDepth);
    }
    return traverse(root);
}
```

## 코드 리뷰
### 정확성
- 모든 테스트케이스 통과. base case와 재귀 로직 정확

### 시간/공간 복잡도
- 시간: O(n) — 모든 노드 한 번씩 방문
- 공간: O(h) — 재귀 호출 스택 (최악 O(n))

### 개선점
- 내부 함수 없이 answer 자체를 재귀로 사용 가능

### 모범 답안
```typescript
function answer(root: TreeNode | null): number {
  if (!root) return 0;
  return 1 + Math.max(answer(root.left), answer(root.right));
}
```

### 핵심 takeaway
트리 재귀의 기본 패턴 — null이면 0, 아니면 1 + max(left, right)

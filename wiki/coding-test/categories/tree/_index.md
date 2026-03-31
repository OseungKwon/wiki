---
category: tree
mastery: 1
total_solved: 1
pass_rate: 100%
last_solved: 2026-03-31
---

## 서브패턴 현황

| 서브패턴 | 수준 | 경험 | pass_rate | 대표 문제 |
|----------|------|------|-----------|----------|
| traversal | F | O | 100% | Maximum Depth of Binary Tree |
| bst | F | X | — | Validate BST |
| lca | F | X | — | Lowest Common Ancestor |
| path-sum | F | X | — | Path Sum |
| construction | I | X | — | Construct from Preorder/Inorder |
| serialization | I | X | — | Serialize and Deserialize |

## 핵심 패턴

### 1. DFS 재귀 (후위 순회)
- 언제: 자식 결과를 합산/비교하여 부모 결과를 구할 때
- 핵심: base case(null → 0 또는 기본값) + 재귀(1 + max/sum of children)
- 템플릿:
```typescript
function dfs(node: TreeNode | null): number {
  if (!node) return 0;
  return 1 + Math.max(dfs(node.left), dfs(node.right));
}
```

## 자주 하는 실수

- [ ] TreeNode는 배열이 아니라 객체 — .length가 아닌 .left/.right로 탐색

## 풀이 목록

| 날짜 | 문제 | 서브패턴 | 난이도 | 결과 | 소요시간 | 링크 |
|------|------|----------|--------|------|----------|------|
| 2026-03-31 | Maximum Depth of Binary Tree | traversal | easy | pass (힌트1) | - | [[2026-03-31-maximum-depth-of-binary-tree]] |

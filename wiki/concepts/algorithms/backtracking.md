---
title: 백트래킹 (Backtracking)
tags:
  - algorithm
  - backtracking
  - recursion
  - typescript
created: 2026-04-15
updated: 2026-04-15
reviewed: false
---

## 정의
선택지를 하나씩 시도하며 재귀 탐색하고, 막히거나 완료되면 되돌아가(pop) 다음 선택지를 탐색하는 알고리즘 패턴.

## 상세 설명

### 핵심 멘탈 모델: 선택의 나무

재귀 함수 호출 하나 = 트리의 노드 하나.  
`push`로 가지를 뻗고, `pop`으로 올라와 옆 가지로 이동한다.

`[1,2,3]` Subsets 탐색 트리:
```
[]
├── [1] → [1,2] → [1,2,3]
│        → [1,3]
├── [2] → [2,3]
└── [3]
```

### 기본 코드 구조

```typescript
function backtrack(index: number, current: number[]) {
    result.push([...current]);     // 현재 상태 저장
    for (let i = index; i < nums.length; i++) {
        current.push(nums[i]);     // 선택 (가지 뻗기)
        backtrack(i + 1, current); // 탐색
        current.pop();             // 선택 취소 (되돌아오기)
    }
}
```

핵심 3줄:
- `push` → 선택
- 재귀 → 탐색
- `pop` → 되돌아옴 (이게 backtracking)

### 언제 쓰는가

| 유형 | 예시 |
|------|------|
| 모든 경우 나열 | 순열, 조합, 부분집합(Subsets) |
| 조건 만족 배치 | N-Queen, 스도쿠 |
| 경로/순서 탐색 | 미로, Word Search |

판단 기준: **"모든 ~를 구하라"** 또는 **"~가 가능한지 판단하라"** + 단계마다 선택지 존재

### 브루트포스 vs 백트래킹

브루트포스는 모든 경우를 다 탐색.  
백트래킹은 불가능한 가지를 일찍 자름(가지치기) → 실질적으로 훨씬 빠름.

DFS를 안다면: **백트래킹 = DFS + 되돌리기**

### 자주 하는 실수: `i+1` vs `index+1`

```typescript
// 틀린 코드
backtrack(index + 1, current);  // index 고정 → 이미 쓴 원소 재방문

// 올바른 코드
backtrack(i + 1, current);      // i 기준 → "현재 선택한 것 다음부터" 보장
```

`index+1`을 쓰면 `i=2`일 때도 항상 `index+1=1`로 돌아가 `nums[1]`, `nums[2]`를 다시 선택 가능.  
결과: `[3,3,3]` 같은 중복 부분집합 생성.

### 중복 원소 허용 문제 (Subsets II)

`nums`에 중복이 있으면 정렬 후 같은 레벨에서 중복 스킵:

```typescript
nums.sort((a, b) => a - b);
// for 루프 안에서:
if (i > index && nums[i] === nums[i - 1]) continue;
```

## 출처 / 참고
- [[concepts/algorithms/binary-search]] — 탐색 알고리즘 비교
- LeetCode 78. Subsets, 90. Subsets II, 46. Permutations

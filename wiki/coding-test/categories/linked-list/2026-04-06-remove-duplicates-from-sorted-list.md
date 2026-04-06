---
date: 2026-04-06
category: linked-list
difficulty: easy
source: LeetCode
result: fail
hints: 3
retries: 0
expected_time: 10~15min
---

## 문제
- **링크**: [LeetCode 83. Remove Duplicates from Sorted List](https://leetcode.com/problems/remove-duplicates-from-sorted-list/)
- **설명**: 정렬된 연결 리스트에서 중복 값 노드를 제거하여 각 값이 한 번만 나타나도록 반환
- **적정 풀이시간**: Easy 10~15분

## 풀이
### 접근 방식
코드 제출 없이 포기. 연결 리스트 포인터 조작 개념 학습 중.

### 코드
```typescript
// 제출 코드 없음 (포기)
```

## 코드 리뷰
### 개선된 코드
```typescript
// 정렬된 리스트이므로 중복은 항상 인접. curr.val === curr.next.val이면 건너뛰기.
function deleteDuplicates(head: ListNode | null): ListNode | null {
  let curr = head;

  while (curr !== null && curr.next !== null) {
    if (curr.val === curr.next.val) {
      curr.next = curr.next.next;  // 중복 노드 건너뛰기
    } else {
      curr = curr.next;            // 다음으로 이동
    }
  }

  return head;
}
```

### 핵심 takeaway
정렬된 리스트에서 중복은 항상 인접해 있으므로, `curr.val === curr.next.val`이면 `curr.next`를 건너뛰면 된다. 같을 때는 curr를 이동하지 않아야 연속 중복(1,1,1)도 처리된다.

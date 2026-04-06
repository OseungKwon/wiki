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
- **링크**: [LeetCode 206. Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/)
- **설명**: 단일 연결 리스트의 head가 주어질 때, 리스트를 뒤집어서 반환
- **적정 풀이시간**: Easy 10~15분

## 풀이
### 접근 방식
배열로 값을 꺼내 reverse하는 방식을 시도했으나, ListNode를 반환해야 하는 요구사항을 충족하지 못함. 포인터 조작 개념(node.next 덮어쓰기)을 이해하지 못해 포기.

### 코드
```typescript
// 제출 코드 없음 (포기)
```

## 코드 리뷰
### 개선된 코드
```typescript
// prev, curr, next 세 포인터로 in-place 뒤집기
function reverseList(head: ListNode | null): ListNode | null {
  let prev: ListNode | null = null;
  let curr = head;

  while (curr !== null) {
    const next = curr.next;  // 다음 노드 임시 저장
    curr.next = prev;        // 방향 뒤집기
    prev = curr;             // prev 전진
    curr = next;             // curr 전진
  }

  return prev;
}
```

### 핵심 takeaway
연결 리스트 조작은 새 노드를 만드는 게 아니라 기존 노드의 `.next` 포인터를 덮어쓰는 것. 다음 노드를 잃지 않도록 임시 변수에 저장하는 것이 핵심.

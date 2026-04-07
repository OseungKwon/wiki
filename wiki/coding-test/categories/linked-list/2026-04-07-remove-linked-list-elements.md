---
date: 2026-04-07
category: linked-list
difficulty: easy
source: LeetCode
result: pass
hints: 0
retries: 1
expected_time: 10~15min
---

## 문제
- **링크**: [LeetCode 203. Remove Linked List Elements](https://leetcode.com/problems/remove-linked-list-elements/)
- **설명**: 연결 리스트에서 특정 값을 가진 모든 노드를 제거하고 새로운 head 반환
- **적정 풀이시간**: Easy 10~15분

## 풀이
### 접근 방식
처음에는 head 특수 케이스를 if문으로 처리하려 했으나, 연속 head 제거 케이스에서 실패. 위키에서 학습한 Dummy Head 패턴을 적용하여 해결.

### 코드
```typescript
function answer(head, val) {
  const dummy = new ListNode(0);
  dummy.next = head;
  let cur = dummy;
  while (cur?.next !== null) {
    if (cur.next.val === val) {
      cur.next = cur.next.next;
    } else {
      cur = cur.next;
    }
  }
  return dummy.next;
}
```

## 코드 리뷰
### 정확성
- 실제 제출: 통과
- 잠재 취약점: 특이사항 없음

### 시간/공간 복잡도
- 시간: O(n) / 공간: O(1)

### 개선된 코드
```typescript
function removeElements(head: ListNode | null, val: number): ListNode | null {
  const dummy = new ListNode(0, head); // 생성자에서 바로 next 연결
  let curr: ListNode | null = dummy;

  while (curr !== null && curr.next !== null) {
    if (curr.next.val === val) {
      curr.next = curr.next.next;
    } else {
      curr = curr.next;
    }
  }

  return dummy.next;
}
```

### 핵심 takeaway
Dummy Head 패턴으로 head 특수 케이스 제거. `dummy.next = head` → 순회 → `return dummy.next`가 공식. head가 제거 대상일 수 있는 문제에서 필수 패턴.

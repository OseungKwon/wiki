---
title: "연결 리스트 (Linked List)"
aliases: [Linked List, ListNode]
tags:
  - data-structure
  - linked-list
  - pointer
  - typescript
created: 2026-04-06
updated: 2026-04-06
reviewed: false
---
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
## 정의
노드들이 `next` 포인터로 체인처럼 연결된 선형 자료구조. 각 노드는 값(`val`)과 다음 노드 참조(`next`)를 가진다.

## 상세 설명

### 참조(Reference) vs 값(Value) — 가장 중요한 개념

JS/TS에서 객체는 참조 타입이다. 변수에 객체를 할당하면 값이 복사되는 게 아니라 **같은 메모리 주소를 가리킨다.**

```typescript
let curr = head;  // curr과 head는 같은 노드를 가리킴
```

| 코드 | 효과 |
|------|------|
| `curr = curr.next` | curr **변수만** 다른 노드로 이동. 실제 구조 변화 없음 |
| `curr.next = X` | **실제 노드의 연결**이 바뀜. head로 봐도 반영됨 |

### 기본 구조

```typescript
class ListNode {
  val: number;
  next: ListNode | null;
  constructor(val: number, next: ListNode | null = null) {
    this.val = val;
    this.next = next;
  }
}
```

- **단방향(Singly)**: `next`만 있음. 한 방향 순회
- **이중(Doubly)**: `prev` + `next`. 양방향 순회

### 핵심 연산

**순회** — `while (curr !== null)` (`curr.val`로 체크하면 val=0에서 멈추는 버그)

**삽입** — 순서가 핵심: 새 노드가 먼저 다음을 가리킨 후, 기존 노드가 새 노드를 가리킴
```typescript
newNode.next = node.next;  // 1. 새 노드 → 다음
node.next = newNode;       // 2. 기존 → 새 노드
```

**삭제** — 건너뛰기: `curr.next = curr.next.next`

**뒤집기** — prev/curr/next 3포인터
```typescript
while (curr !== null) {
  const next = curr.next;  // 임시 저장 (필수!)
  curr.next = prev;        // 방향 뒤집기
  prev = curr;
  curr = next;
}
return prev;
```

### 주요 패턴

| 패턴 | 용도 | 핵심 |
|------|------|------|
| prev/curr/next | 뒤집기, 삭제, 방향 변경 | next를 먼저 저장 |
| Fast-Slow 포인터 | 중간 찾기, 사이클 감지 | slow 1칸, fast 2칸 |
| Dummy Head | head가 바뀔 수 있는 삽입/삭제 | `dummy.next = head` |
| In-place 수정 | 메모리 효율적 조작 | 새 노드 안 만들고 포인터만 조작 |

### 흔한 실수

1. `next` 저장 안 하고 포인터 변경 → 연결 끊김
2. `while (curr.next)` vs `while (curr)` 혼동
3. `while (curr.val)` → val=0이면 조기 종료
4. 빈 리스트 / 단일 노드 처리 누락
5. 삽입 시 순서 실수 → 기존 연결 소실

### 배열 vs 연결 리스트

| 연산 | 배열 | 연결 리스트 |
|------|------|------------|
| 인덱스 접근 | O(1) | O(n) |
| 맨 앞 삽입/삭제 | O(n) | O(1) |
| 중간 삽입/삭제 | O(n) | O(1) (위치 알 때) |
| 검색 | O(n) | O(n) |

### 코딩 테스트 빈출 유형

**Easy**: 뒤집기(206), 중간 노드(876), 합치기(21), 사이클(141), 중복 제거(83)
**Medium**: N번째 삭제(19), 팰린드롬(234), 두 수 더하기(2), 사이클 시작점(142)
**Hard**: K개씩 뒤집기(25), 리스트 정렬(148)

## 출처 / 참고
- [[binary-search|Binary Search]] — 같은 알고리즘/자료구조 카테고리
- [[bucket-sort|Bucket Sort]] — 같은 알고리즘/자료구조 카테고리

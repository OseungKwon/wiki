---
title: 힙 (Heap)
aliases:
  - heap
  - 최소 힙
  - 최대 힙
  - Min Heap
  - Max Heap
tags:
  - data-structure
  - heap
  - priority-queue
  - algorithm
  - javascript
created: 2026-04-15
updated: 2026-04-15
reviewed: false
---

## 정의
완전 이진 트리(Complete Binary Tree) 기반의 자료구조. 부모-자식 간 우선순위 관계를 항상 유지하며, 최솟값/최댓값을 O(1)로 조회하고 O(log n)으로 삽입/추출할 수 있다.

- **최소 힙(Min Heap)**: 부모 ≤ 자식 (루트가 최솟값)
- **최대 힙(Max Heap)**: 부모 ≥ 자식 (루트가 최댓값)

## 상세 설명

### 배열로 표현

트리 구조를 1차원 배열로 표현한다. 인덱스 0을 루트로 사용:

```
- 부모:        Math.floor((i - 1) / 2)
- 왼쪽 자식:   2 * i + 1
- 오른쪽 자식: 2 * i + 2
```

```
        10
       /  \
      7    8
     / \  /
    3   4 5

배열: [10, 7, 8, 3, 4, 5]
```

### 최소 힙 구현 (JavaScript)

```javascript
class MinHeap {
  constructor() { this.heap = []; }

  parent(i) { return Math.floor((i - 1) / 2); }
  left(i)   { return 2 * i + 1; }
  right(i)  { return 2 * i + 2; }
  swap(i, j) { [this.heap[i], this.heap[j]] = [this.heap[j], this.heap[i]]; }

  push(val) {
    this.heap.push(val);
    this._bubbleUp(this.heap.length - 1);
  }

  _bubbleUp(i) {
    while (i > 0) {
      const p = this.parent(i);
      if (this.heap[p] > this.heap[i]) { this.swap(p, i); i = p; }
      else break;
    }
  }

  peek() { return this.heap[0]; }

  pop() {
    if (this.heap.length === 1) return this.heap.pop();
    const min = this.heap[0];
    this.heap[0] = this.heap.pop();
    this._bubbleDown(0);
    return min;
  }

  _bubbleDown(i) {
    const n = this.heap.length;
    while (true) {
      let smallest = i;
      const l = this.left(i), r = this.right(i);
      if (l < n && this.heap[l] < this.heap[smallest]) smallest = l;
      if (r < n && this.heap[r] < this.heap[smallest]) smallest = r;
      if (smallest !== i) { this.swap(i, smallest); i = smallest; }
      else break;
    }
  }

  size() { return this.heap.length; }
}
```

### 최대 힙 전환

비교 방향만 뒤집는다:
- `_bubbleUp`: `heap[p] > heap[i]` → `heap[p] < heap[i]`
- `_bubbleDown`: `heap[l] < heap[smallest]` → `heap[l] > heap[smallest]`

### 시간 복잡도

| 연산 | 복잡도 |
|------|--------|
| push | O(log n) |
| pop  | O(log n) |
| peek | O(1) |
| heapify (배열 → 힙) | O(n) |

### 힙을 사용하는 이유

배열(미정렬)은 삽입 O(1)이지만 최솟값 탐색 O(n), 정렬 배열은 삽입 O(n)이 느림. 힙은 **삽입과 추출 모두 O(log n)으로 균형**이 잡혀 있다.

핵심: "루트만 정렬되어 있으면 된다"는 느슨한 조건 덕분에 매번 전체 정렬 없이 빠르게 동작한다.

### 주요 활용

| 사례 | 설명 |
|------|------|
| 우선순위 큐 | 작업 스케줄링, 이벤트 처리 |
| 다익스트라 | 최단 거리 노드 O(log n) 추출 → O((V+E) log V) |
| Top-K 문제 | 크기 K짜리 힙만 유지 → O(n log k) |
| 실시간 중앙값 | 최대힙(하위 절반) + 최소힙(상위 절반) 두 개 조합 |

## 출처 / 참고
- [[linked-list]] — 같은 data-structure 카테고리

---
date: 2026-04-07
category: greedy
difficulty: easy
source: LeetCode
result: pass
hints: 0
retries: 2
expected_time: 10~15min
---

## 문제
- **링크**: [LeetCode 455. Assign Cookies](https://leetcode.com/problems/assign-cookies/)
- **설명**: 아이들의 요구 크기와 쿠키 크기가 주어질 때, 만족하는 아이의 수를 최대화
- **적정 풀이시간**: Easy 10~15분

## 풀이
### 접근 방식
양쪽 배열을 정렬 후, 가장 큰 요구부터 가장 큰 쿠키를 탐욕적으로 매칭. 매칭 가능하면 쿠키를 pop하고 카운트 증가.

### 코드
```typescript
function findContentChildren(g: number[], s: number[]): number {
  g.sort((a, b) => a - b);
  s.sort((a, b) => a - b);
  let cnt = 0;
  for (let i = 0; i < g.length; i++) {
    if (s.length === 0) return cnt;
    if (g[g.length - 1 - i] <= s[s.length - 1]) {
      s.pop();
      cnt++;
    }
  }
  return cnt;
}
```

## 코드 리뷰
### 정확성
- 실제 제출: 통과 (2차, 1차는 `.sort()` 사전순 정렬 버그)

### 시간/공간 복잡도
- 시간: O(n log n) / 공간: O(1)

### 개선된 코드
```typescript
// 동일 접근법, 타입 명시 버전
function findContentChildren(g: number[], s: number[]): number {
  g.sort((a, b) => a - b);
  s.sort((a, b) => a - b);
  let cnt = 0;
  for (let i = 0; i < g.length; i++) {
    if (s.length === 0) return cnt;
    if (g[g.length - 1 - i] <= s[s.length - 1]) {
      s.pop();
      cnt++;
    }
  }
  return cnt;
}
```

### 핵심 takeaway
JS `.sort()`는 기본 사전순 정렬. 숫자 정렬은 반드시 `(a, b) => a - b` 비교 함수 필요. Greedy 문제는 "정렬 후 탐욕적 매칭"이 기본 패턴.

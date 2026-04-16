---
date: 2026-04-16
category: math-bit
difficulty: easy
source: LeetCode
result: pass
hints: 0
retries: 2
expected_time: 10~15min
---

## 문제
- **링크**: https://leetcode.com/problems/number-of-1-bits/
- **설명**: 양의 정수 n의 이진수 표현에서 1인 비트의 개수(Hamming weight)를 반환하라.
- **적정 풀이시간**: 10~15min

## 풀이
### 접근 방식
이진수 문자열로 변환 후 '1'의 개수를 카운트.
(첫 시도는 문자열만 반환 → filter로 카운팅 추가 후 통과)

### 코드
```typescript
function answer(num: number): number {
    let result = '';
    while (num > 0) {
        result = (num % 2) + result;
        num = num >> 1;
    }
    return result.split('').filter(s => s === '1').length;
}
```

## 코드 리뷰
### 정확성
- 실제 제출: 통과
- 잠재 취약점: 특이사항 없음

### 시간/공간 복잡도
- 시간: O(log n), 공간: O(log n) (문자열 생성)
- 개선: 문자열 없이 O(1) 공간 가능

### 개선된 코드
```typescript
// 방법 1: 직접 카운팅 O(log n), O(1)
function hammingWeight(n: number): number {
    let count = 0;
    while (n > 0) {
        count += n & 1;
        n >>>= 1;
    }
    return count;
}

// 방법 2: n & (n-1) 트릭 — 1의 개수만큼만 반복
function hammingWeight(n: number): number {
    let count = 0;
    while (n !== 0) {
        n &= n - 1;  // 가장 오른쪽 1 제거
        count++;
    }
    return count;
}
```

### 핵심 takeaway
`n & 1`로 최하위 비트 확인, `n & (n-1)`으로 최하위 1 제거 — 비트 문제의 표준 트릭

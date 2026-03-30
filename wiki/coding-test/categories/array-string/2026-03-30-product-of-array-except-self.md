---
date: 2026-03-30
category: array-string
difficulty: medium
source: LeetCode
result: pass
hint: true
time: -
---

## 문제
- **링크**: https://leetcode.com/problems/product-of-array-except-self
- **설명**: 정수 배열에서 자기 자신을 제외한 나머지 모든 원소의 곱 배열을 반환. 나눗셈 사용 금지, O(n).

## 풀이
### 접근 방식
Prefix/Suffix 누적곱 패턴. 왼쪽 누적곱 배열과 오른쪽 누적곱 배열을 각각 구한 뒤 곱해서 결과를 만듦. (힌트 참고)

### 코드
```javascript
function answer(arr) {
    const leftAccArr = []
    const rightAccArr = []

    arr.reduce( (acc, cur, index) => {
        leftAccArr.push(acc)
        return acc * cur
    }, 1)

    arr.reduce( (acc, cur, index) => {
        rightAccArr.unshift(acc)
        return acc * arr[arr.length - index - 1]
    }, 1)

    return rightAccArr.map( (cur, index) => cur * leftAccArr[index])
}
```

## 코드 리뷰
### 정확성
로직 정확. 모든 케이스(0 포함, 음수) 정상 동작.

### 시간/공간 복잡도
- 시간: O(n²) — `unshift`가 루프 안에서 O(n)
- 공간: O(n) — leftAccArr, rightAccArr 별도 생성

### 개선점
1. `unshift` → index 기반 할당 또는 `reduceRight` 사용 (O(n²) → O(n))
2. TypeScript 타입 추가
3. 공간 최적화: output 배열 하나로 두 패스 처리 가능

### 모범 답안
```typescript
function productExceptSelf(nums: number[]): number[] {
  const n = nums.length;
  const answer: number[] = new Array(n);
  answer[0] = 1;
  for (let i = 1; i < n; i++) {
    answer[i] = answer[i - 1] * nums[i - 1];
  }
  let right = 1;
  for (let i = n - 1; i >= 0; i--) {
    answer[i] *= right;
    right *= nums[i];
  }
  return answer;
}
```

### 핵심 takeaway
`unshift`는 O(n)이므로 루프 안에서 피할 것. 누적곱은 양방향 단일 패스로 O(1) 공간에 풀 수 있다.

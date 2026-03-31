---
date: 2026-03-31
category: stack-queue
difficulty: easy
source: LeetCode
result: pass
hints: 0
retries: 2
expected_time: 10~15min
---

## 문제
- **링크**: https://leetcode.com/problems/valid-parentheses/
- **설명**: 괄호 문자열이 유효한지 판별 (3종류 괄호)
- **적정 풀이시간**: 10~15분

## 풀이
### 접근 방식
괄호 종류별 preset 배열을 만들어 여는 괄호는 스택에 push, 닫는 괄호는 스택 top과 비교하여 매칭 확인.

### 코드
```typescript
function answer(s: string): boolean {
    const stack: string[] = [];
    const preset = [
        ['(', ')'],
        ['[', ']'],
        ['{', '}'],
    ];
    for (let i = 0; i < s.length; i++) {
        for (let j = 0; j < preset.length; j++) {
            if (s[i] === preset[j][0]) {
                stack.push(s[i]);
            }
            if (s[i] === preset[j][1]) {
                if (stack[stack.length - 1] === preset[j][0]) {
                    stack.pop();
                } else {
                    return false;
                }
            }
        }
    }
    return stack.length === 0;
}
```

## 코드 리뷰
### 정확성
- 실제 제출: 통과
- 잠재 취약점: 특이사항 없음

### 시간/공간 복잡도
- 시간: O(n) / 공간: O(n)
- 이미 최적해

### 개선점
1. Map으로 닫는→여는 매핑하면 내부 루프 제거 가능
2. for...of로 순회 간결화
3. 첫 버전 forEach 안의 return 스코프 버그 주의

### 개선된 코드
```typescript
function isValid(s: string): boolean {
  const map = new Map<string, string>([
    [')', '('],
    [']', '['],
    ['}', '{'],
  ]);
  const stack: string[] = [];

  for (const ch of s) {
    if (map.has(ch)) {
      if (stack.pop() !== map.get(ch)) return false;
    } else {
      stack.push(ch);
    }
  }

  return stack.length === 0;
}
```

### 핵심 takeaway
forEach 콜백 안의 return은 외부 함수를 종료하지 않는다. 조기 종료가 필요하면 for 루프를 사용할 것.

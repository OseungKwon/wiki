---
date: 2026-03-30
category: array-string
difficulty: medium
source: LeetCode
result: pass
hints: 0
retries: 1
expected_time: 20~35min
retry_from_keep: false
---

## 문제
- **링크**: https://leetcode.com/problems/group-anagrams
- **설명**: 문자열 배열에서 애너그램끼리 그룹화하여 반환.

## 풀이
### 접근 방식
각 문자열을 정렬해 키로 사용, Map에 같은 키를 가진 원본 문자열들을 그룹핑.

### 코드
```javascript
function answer(strs) {
    strs = strs.map(str => ({
        origin: str,
        compare: str.split('').sort().join('')
    }))
    const map = new Map()
    strs.map(str => {
        if (map.has(str.compare)) {
            map.set(str.compare, [...map.get(str.compare), str.origin])
        } else {
            map.set(str.compare, [str.origin])
        }
    })
    const result = []
    map.forEach((value, key) => {
        result.push(value)
    })
    return result;
}
```

## 코드 리뷰
### 정확성
로직 정확. 모든 케이스 정상 동작.

### 시간/공간 복잡도
- 시간: O(n * k log k) — 최적
- 공간: O(n * k)

### 개선점
1. `.map()`을 부수효과용으로 사용 → `forEach` 사용할 것
2. 중간 객체 `{origin, compare}` 불필요 — 한 번의 순회로 처리 가능
3. `map.forEach + push` → `[...map.values()]`로 간소화
4. `map.get() ?? []`로 if/else 분기 제거

### 개선된 코드
```typescript
// 개선: .map→for-of, 중간 객체 제거, map.get() ?? []로 분기 제거, [...map.values()]로 간소화
function groupAnagrams(strs: string[]): string[][] {
  const map = new Map<string, string[]>();
  for (const str of strs) {
    const key = str.split('').sort().join('');
    const group = map.get(key) ?? [];
    group.push(str);
    map.set(key, group);
  }
  return [...map.values()];
}
```

### 핵심 takeaway
Map + 정렬 키 = 그룹핑 패턴. `.map()`은 반환값 쓸 때만, 부수효과는 `forEach`. `[...map.values()]`로 간결 변환.

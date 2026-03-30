---
category: array-string
mastery: 2
total_solved: 2
pass_rate: 100%
last_solved: 2026-03-30
---

## 핵심 패턴
## 서브패턴 현황

| 서브패턴 | 수준 | 경험 | pass_rate | 대표 문제 |
|----------|------|------|-----------|----------|
| In-place manipulation | F | X | — | Remove Duplicates from Sorted Array |
| Prefix Sum | F | O | 100% | Product of Array Except Self |
| Kadane's Algorithm | F | X | — | Maximum Subarray |
| Dutch National Flag | F | X | — | Sort Colors |
| Rotate / Reverse | F | X | — | Rotate Array |
| Boyer-Moore Voting | I | X | — | Majority Element |
| Difference Array | I | X | — | Range Addition |
| Interval Merge / Sweep Line | I | X | — | Merge Intervals |
| Matrix Traversal | I | X | — | Spiral Matrix |
| String Matching (KMP/Rabin-Karp) | A | X | — | Implement strStr() |


### 1. Prefix/Suffix 누적 (곱, 합)
- 언제: 자기 자신을 제외한 나머지 원소의 연산 결과가 필요할 때
- 핵심: 왼쪽 누적 + 오른쪽 누적을 따로 구한 뒤 합침
- 템플릿:
```typescript
answer[0] = 1;
for (let i = 1; i < n; i++) {
  answer[i] = answer[i - 1] * nums[i - 1];
}
let right = 1;
for (let i = n - 1; i >= 0; i--) {
  answer[i] *= right;
  right *= nums[i];
}
```
- 공간 최적화: 별도 배열 대신 output 배열 재활용 → O(1)

### 2. 정렬 키 + Map 그룹핑
- 언제: 동일한 구성 요소를 가진 항목끼리 묶을 때 (애너그램, 중복 탐지 등)
- 핵심: 각 항목을 정규화(정렬 등)해서 Map의 키로 사용, 같은 키끼리 그룹
- 템플릿:
```typescript
const map = new Map<string, string[]>();
for (const str of strs) {
  const key = str.split('').sort().join('');
  const group = map.get(key) ?? [];
  group.push(str);
  map.set(key, group);
}
return [...map.values()];
```
- `map.get() ?? []`로 if/else 분기 제거

## 자주 하는 실수

- [ ] `Array.unshift()`를 루프 안에서 사용 → O(n²). index 기반 할당이나 reverse 사용할 것
- [ ] reduce로 역순 처리 시 `reduceRight` 대신 index 계산 → 가독성 저하
- [ ] `.map()`을 부수효과용으로 사용 → 반환값 안 쓰면 `forEach` 사용할 것
- [ ] Map → Array 변환 시 수동 push → `[...map.values()]`로 간소화

## 풀이 목록
## 풀이 목록

| 날짜 | 문제 | 서브패턴 | 난이도 | 결과 | 소요시간 | 링크 |
|------|------|----------|--------|------|----------|------|
| 2026-03-30 | Product of Array Except Self | Prefix Sum | medium | pass (힌트) | - | [[2026-03-30-product-of-array-except-self]] |
| 2026-03-30 | Group Anagrams | Grouping/Bucketing | medium | pass | - | [[2026-03-30-group-anagrams]] |

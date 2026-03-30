---
category: array-string
mastery: 2
total_solved: 1
pass_rate: 100%
last_solved: 2026-03-30
---

## 핵심 패턴

### 1. Prefix/Suffix 누적 (곱, 합)
- 언제: 자기 자신을 제외한 나머지 원소의 연산 결과가 필요할 때
- 핵심: 왼쪽 누적 + 오른쪽 누적을 따로 구한 뒤 합침
- 템플릿:
```typescript
// 왼쪽 누적을 answer에 직접 저장
answer[0] = 1;
for (let i = 1; i < n; i++) {
  answer[i] = answer[i - 1] * nums[i - 1];
}
// 오른쪽 누적을 곱하면서 완성
let right = 1;
for (let i = n - 1; i >= 0; i--) {
  answer[i] *= right;
  right *= nums[i];
}
```
- 공간 최적화: 별도 배열 대신 output 배열 재활용 → O(1)

## 자주 하는 실수

- [ ] `Array.unshift()`를 루프 안에서 사용 → O(n²). index 기반 할당이나 reverse 사용할 것
- [ ] reduce로 역순 처리 시 `reduceRight` 대신 index 계산 → 가독성 저하

## 풀이 목록

| 날짜 | 문제 | 난이도 | 결과 | 소요시간 | 링크 |
|------|------|--------|------|----------|------|
| 2026-03-30 | Product of Array Except Self | medium | pass (힌트) | - | [[2026-03-30-product-of-array-except-self]] |

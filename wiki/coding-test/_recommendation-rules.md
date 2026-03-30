---
version: 2.0
last_updated: 2026-03-30
---

# 문제 추천 알고리즘 규칙

> `/ct start` 실행 시 이 규칙을 순서대로 적용한다.

---

## Rule 1. 카테고리 연속 제한 (Interleaving)

**근거**: Rohrer, D. & Taylor, K. (2007). "The shuffling of mathematics problems improves learning." *Instructional Science*, 35(6), 481-498; Kornell, N. & Bjork, R. A. (2008). "Learning concepts and categories." *Psychological Science*, 19(6), 585-592.

- **신규 카테고리** (해당 카테고리 풀이 < 3): 최대 **3회** 연속 허용
- **기존 카테고리** (해당 카테고리 풀이 >= 3): 최대 **2회** 연속 허용
- 연속 제한 도달 후 최소 **2문제**는 다른 카테고리를 풀어야 다시 추천 가능
- `_log.md`의 최근 기록으로 연속 횟수를 판단한다

---

## Rule 2. 난이도 조절 (85% Rule)

**근거**: Wilson, R. C., Shenhav, A., Stiso, J. & Cohen, J. D. (2019). "The Eighty Five Percent Rule for Optimal Learning." *Nature Communications*, 10, 4646.

카테고리별 **최근 풀이 기준** (최소 3문제 이상일 때 적용):

| 최근 성공률 | 행동 |
|------------|------|
| > 90% | 난이도 1단계 상향 (Rule 6 체계에 따름) |
| **80~90%** | **유지 (최적 구간)** |
| 70~80% | 난이도 유지, 유사 패턴 반복 |
| < 70% | 난이도 1단계 하향 + 기초 패턴 복습 |

- 성공률 계산 시 최근 **20문제** 이동 윈도우 사용 (카테고리 내)
- 데이터 부족 시(< 3문제) Easy~Medium으로 시작

---

## Rule 3. 카테고리 선택 점수 (Priority Score)

**근거**: Ericsson, K. A., Krampe, R. T. & Tesch-Römer, C. (1993). "The role of deliberate practice in the acquisition of expert performance." *Psychological Review*, 100(3), 363-406; Wozniak, P. (1990). "Optimization of Learning." SM-2 Algorithm, SuperMemo; Rohrer & Taylor (2007), 상동.

```
priority = (0.4 × weakness) + (0.35 × freshness) + (0.25 × diversity)
```

| 요소 | 산출 방식 | 범위 | 근거 |
|------|----------|------|------|
| weakness | `(100 - pass_rate) / 100` | 0~1 | Ericsson: 연습의 40%를 약점에 |
| freshness | `min(경과일수, 14) / 14` | 0~1 | SM-2: 3차 복습 시점 ~14일 |
| diversity | `1 - (카테고리 풀이수 / 총 풀이수)` | 0~1 | Rohrer: 인터리빙 전이 학습 |

- Rule 1에 의해 제외된 카테고리는 점수 계산에서 **스킵**
- 동점 시 freshness가 높은 카테고리 우선

---

## Rule 4. 단계별 전략 (ZPD 기반)

**근거**: Vygotsky, L. S. (1978). "Mind in Society: The Development of Higher Psychological Processes." Harvard University Press; Kornell & Bjork (2008), 상동.

| 단계 | 조건 | 전략 |
|------|------|------|
| **탐색기** | 경험 카테고리 < 5개 | 미경험 카테고리를 블로킹으로 소개 (2~3문제), Easy~Medium |
| **강화기** | 경험 카테고리 >= 5개 | Rule 3 priority score 기반 인터리빙, Rule 2로 난이도 조절 |
| **마스터기** | 평균 mastery >= 3 & 경험 >= 10개 | 복합 문제(2개 카테고리 결합), Hard 비중 확대 |

---

## Rule 5. 예외 처리 (Rule 1에 종속)

**근거**: Wozniak, P. (1990). SM-2 Algorithm (fail 시 간격 리셋); Leitner, S. (1972). "So lernt man lernen." (오답 시 Box 1 리셋)

| 상황 | Rule 1 미발동 (연속 < 제한) | Rule 1 발동 (연속 = 제한) |
|------|---------------------------|--------------------------|
| 연속 pass | 난이도 상향, 같은 카테고리 유지 | 카테고리 변경, 복귀 시 상향 난이도 적용 |
| 연속 fail | 난이도 하향 + 기초 복습 | 카테고리 변경, 복귀 시 하향 난이도 적용 |

- Rule 1이 **항상 우선**
- 난이도 조정은 즉시 `_index.md`의 mastery에 반영 → 복귀 시 자동 적용
- fail 후 **1~3일 내 유사 문제** 재출제 (SM-2 lapse 처리)

---

## Rule 6. 난이도 체계 (Difficulty Classification)

**근거**: Ihantola, P. et al. (2015). "Educational data mining and learning analytics in programming." *ITiCSE WGR*, 41-63 (개념 수가 난이도의 최강 예측 인자); Pelánek, R. (2016). "Applications of the Elo rating system in adaptive educational systems." *Computers & Education*, 98, 169-179; Fuller, U. et al. (2007). "Developing a computer science-specific learning taxonomy." *ACM SIGCSE Bulletin*, 39(4), 152-170 (Bloom's Taxonomy for CS); LeetCode/Codeforces/프로그래머스 플랫폼 데이터.

### 난이도 정의

| 난이도 | 필요 개념 수 | 예상 풀이 시간 | 서브패턴 수준 | 최적화 요구 | 프로그래머스 | Codeforces |
|--------|------------|--------------|-------------|-----------|------------|-----------|
| **Easy** | 1개 | 10~15분 | Foundational | 브루트포스 OK | Lv.1 | 800~1200 |
| **Medium** | 2~3개 | 20~35분 | Intermediate | 브루트포스→최적화 필요 | Lv.2~3 | 1200~1800 |
| **Hard** | 3개+ | 40~60분+ | Advanced | 비자명한 최적화 필수 | Lv.3~4 | 1800+ |

### 난이도 판정 기준 (우선순위 순)

1. **필요 개념 수** — 가장 강한 예측 인자 (Ihantola et al. 2015)
2. **서브패턴 수준** — Rule 7의 Foundational/Intermediate/Advanced에 매핑
3. **복잡도 갭** — 브루트포스와 최적해 사이의 시간 복잡도 차이
4. **엣지케이스 밀도** — 경계값 처리, 빈 입력, 오버플로우 등 특수 케이스 수
5. **비자명성** — 숨겨진 관찰, 패턴 전환, 수학적 통찰 필요 여부

### Rule 2와의 연동

Rule 2(85% Rule)에서 "난이도 1단계 상향/하향"은 이 체계에 따른다:
- 상향: Easy → Medium → Hard
- 하향: Hard → Medium → Easy
- 같은 난이도 내에서도 서브패턴 수준으로 미세 조절 (Foundational → Intermediate → Advanced)

---

## Rule 7. 카테고리 내 서브패턴 추천 (Sub-pattern Diversity)

**근거**: Design Gurus, "Grokking the Coding Interview" (서브패턴 분류 체계); NeetCode Roadmap (카테고리별 학습 경로); Blind 75 / Grind 75, Sean Prasad (핵심 문제 분류); LeetCode Official Study Plans (패턴 태그); Ericsson et al. (1993), 상동 (deliberate practice — 약점 서브패턴 집중).

각 카테고리는 Foundational(F) / Intermediate(I) / Advanced(A) 수준의 서브패턴으로 구성된다. `_index.md`의 서브패턴 테이블에서 경험 여부를 추적한다.

전체 서브패턴 목록은 각 카테고리의 `_index.md` 서브패턴 섹션에 기재한다.

### 서브패턴 추천 정책

1. **미경험 서브패턴 우선**: `_index.md`의 서브패턴 테이블에서 경험 없는 패턴을 우선 추천
2. **수준 순서 준수**: 같은 카테고리 내에서 Foundational → Intermediate → Advanced 순서로 경험
3. **fail한 서브패턴 재도전**: pass_rate가 낮은 서브패턴의 유사 문제를 우선 배치
4. **Rule 6과 연동**: 서브패턴 수준(F/I/A)이 난이도(Easy/Medium/Hard)와 대응

### 서브패턴 수 요약

| 카테고리 | 총 | F | I | A |
|----------|---|---|---|---|
| array-string | 10 | 5 | 3 | 2 |
| hash-map | 6 | 3 | 1 | 2 |
| two-pointer | 7 | 4 | 3 | 0 |
| stack-queue | 6 | 2 | 3 | 1 |
| binary-search | 5 | 2 | 3 | 0 |
| dynamic-programming | 14 | 4 | 6 | 4 |
| graph | 11 | 3 | 6 | 2 |
| tree | 9 | 2 | 5 | 2 |
| greedy | 7 | 3 | 3 | 1 |
| backtracking | 5 | 2 | 3 | 0 |
| heap | 5 | 1 | 4 | 0 |
| sorting | 5 | 1 | 4 | 0 |
| **합계** | **~90** | **~32** | **~44** | **~14** |

---

## 적용 순서 요약

```
1. _log.md에서 최근 풀이 기록 로드
2. Rule 1 적용 → 연속 제한에 걸린 카테고리 제외
3. Rule 4 적용 → 현재 단계 판별 (탐색기/강화기/마스터기)
4. Rule 3 적용 → 후보 카테고리별 priority score 산출, 최고 점수 카테고리 선택
5. Rule 6 적용 → 난이도 체계 기준으로 문제 난이도 범위 설정
6. Rule 2 적용 → 성공률 기반 난이도 미세 조절
7. Rule 7 적용 → 선택된 카테고리 내 미경험/약점 서브패턴 우선 선택
8. Rule 5 확인 → 연속 pass/fail 예외 처리
9. 문제 출제
```

---

## 참고 문헌

1. Wilson, R. C., Shenhav, A., Stiso, J. & Cohen, J. D. (2019). "The Eighty Five Percent Rule for Optimal Learning." *Nature Communications*, 10, 4646.
2. Rohrer, D. & Taylor, K. (2007). "The shuffling of mathematics problems improves learning." *Instructional Science*, 35(6), 481-498.
3. Kornell, N. & Bjork, R. A. (2008). "Learning concepts and categories: Is spacing the enemy of induction?" *Psychological Science*, 19(6), 585-592.
4. Ericsson, K. A., Krampe, R. T. & Tesch-Römer, C. (1993). "The role of deliberate practice in the acquisition of expert performance." *Psychological Review*, 100(3), 363-406.
5. Wozniak, P. (1990). "Optimization of Learning." SM-2 Algorithm, SuperMemo.
6. Leitner, S. (1972). "So lernt man lernen."
7. Vygotsky, L. S. (1978). "Mind in Society: The Development of Higher Psychological Processes." Harvard University Press.
8. Ihantola, P. et al. (2015). "Educational data mining and learning analytics in programming: Literature review and case studies." *ITiCSE Working Group Reports*, 41-63.
9. Pelánek, R. (2016). "Applications of the Elo rating system in adaptive educational systems." *Computers & Education*, 98, 169-179.
10. Fuller, U. et al. (2007). "Developing a computer science-specific learning taxonomy." *ACM SIGCSE Bulletin*, 39(4), 152-170.
11. Design Gurus. "Grokking the Coding Interview: Patterns for Coding Questions."
12. NeetCode Roadmap. https://neetcode.io/roadmap
13. Blind 75 / Grind 75 (Sean Prasad, Yangshun Tay). Curated coding interview problem lists.

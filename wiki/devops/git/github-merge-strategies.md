---
title: GitHub Merge 전략
aliases:
  - merge commit
  - squash and merge
  - rebase and merge
tags:
  - git
  - github
  - merge
  - pull-request
  - workflow
created: 2026-03-18
updated: 2026-03-18
cssclasses:
---

## 정의

GitHub PR을 병합할 때 선택할 수 있는 3가지 merge 전략. 각 전략은 커밋 히스토리의 형태와 보존 방식이 다르다.

## 상세 설명

### Merge Commit (Create a merge commit)

- `git merge --no-ff`와 동일
- 모든 커밋 이력을 보존하고 별도의 merge commit을 생성
- 히스토리가 비선형(분기 형태)이 되지만 전체 이력 추적 가능

```
*   merge commit (main)
|\
| * feat: B
| * feat: A
|/
* previous commit
```

### Squash and Merge

- PR의 모든 커밋을 **하나의 커밋으로 압축**하여 base branch에 추가
- 개별 커밋 이력은 사라지지만 깔끔한 선형 히스토리 유지
- "wip", "fix typo" 같은 중간 커밋이 많을 때 유용

```
* squashed: feat A + B (main)
* previous commit
```

### Rebase and Merge

- PR의 커밋들을 base branch 위에 **재배치(rebase)**
- merge commit 없이 선형 히스토리 유지
- 각 커밋이 개별적으로 보존됨

```
* feat: B (main)
* feat: A
* previous commit
```

## 비교

| 방식 | merge commit | 커밋 보존 | 히스토리 |
|------|-------------|----------|---------|
| Merge Commit | O | O | 비선형 |
| Squash | X | X (1개로 압축) | 선형 |
| Rebase | X | O | 선형 |

## 실무 적용

- **Squash**: 소규모 feature PR에서 가장 보편적. main 히스토리가 깔끔해짐
- **Merge Commit**: 큰 feature나 release branch 병합 시. 분기 이력이 필요할 때
- **Rebase**: 선형 히스토리를 선호하는 팀. 각 커밋이 의미 있을 때

## 출처 / 참고

- [GitHub Docs - Merging a pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/incorporating-changes-from-a-pull-request/about-pull-request-merges)
- [[stacked-prs-rebase-onto|Git 스택 PR - rebase --onto]]
- [[rebase-deep-dive|Git Rebase 심화 가이드]]

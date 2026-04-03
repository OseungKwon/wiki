---
title: Git Rebase 심화 가이드
aliases:
  - git rebase
  - rebase --onto
  - interactive rebase
tags:
  - git
  - rebase
  - workflow
created: 2026-03-18
updated: 2026-03-18
reviewed: false
---

## 정의

`git rebase`는 커밋 히스토리를 재배치하여 선형 히스토리를 만드는 명령어다. merge와 달리 merge commit 없이 커밋들을 대상 브랜치 위에 다시 쌓는다.

## 상세 설명

### 기본 사용법

```bash
git rebase main              # main 위에 현재 브랜치 커밋 재배치
git rebase -i HEAD~3         # 최근 3개 커밋을 인터랙티브하게 편집
```

### 인터랙티브 rebase 명령어

| 명령 | 설명 |
|------|------|
| `pick` | 커밋 그대로 사용 |
| `reword` | 커밋 메시지 수정 |
| `squash` | 이전 커밋과 합침 (메시지 편집) |
| `fixup` | 이전 커밋과 합침 (메시지 버림) |
| `drop` | 커밋 삭제 |

### rebase --onto

가장 강력하면서 헷갈리는 옵션. 핵심 문법:

```bash
git rebase --onto <new-base> <old-base> <branch>
```

**"branch에서 old-base 이후의 커밋들만 떼어내서 new-base 위에 붙여라"**

#### 시나리오 1: 브랜치의 기반 변경

```
A - B - C (main)
         \
          D - E (feature)
               \
                F - G (sub-feature)
```

`sub-feature`를 `main` 위에 직접 붙이기:

```bash
git rebase --onto main feature sub-feature
```

```
A - B - C (main)
         \       \
          \       F' - G' (sub-feature)
           \
            D - E (feature)
```

#### 시나리오 2: 중간 커밋 제거

```
A - B - C - D - E (feature)
```

C, D를 제거하고 B 위에 E를 올리기:

```bash
git rebase --onto B D feature
```

```
A - B - E' (feature)
```

#### 시나리오 3: 다른 브랜치로 이식

```
A - B (main)
     \
      C - D (release)
           \
            E - F (my-feature)
```

`release` 기반을 `main` 기반으로 변경:

```bash
git rebase --onto main release my-feature
```

```
A - B (main)
     \       \
      \       E' - F' (my-feature)
       \
        C - D (release)
```

### 심화 옵션

| 옵션 | 설명 |
|------|------|
| `--keep-base` | 기반은 유지하고 커밋만 정리. `-i`와 조합 시 유용 |
| `--autosquash` | `fixup!`/`squash!` 접두사 커밋을 자동 배치. 코드 리뷰 후 수정 반영에 유용 |
| `--autostash` | dirty 워킹 디렉토리에서도 자동 stash → rebase → pop |
| `--rebase-merges` | merge 커밋 보존하면서 rebase (`--preserve-merges` 대체) |

**autosquash 워크플로우:**

```bash
git commit --fixup=<commit-hash>    # fixup 커밋 생성
git rebase -i --autosquash main     # 자동으로 해당 커밋 아래에 배치
```

### 충돌 처리

```bash
# 충돌 해결 후 계속
git add <resolved-files>
git rebase --continue

# rebase 중단, 원래 상태로 복원
git rebase --abort
```

### 위험 상황 복구

```bash
# reflog에서 rebase 이전 상태 찾기
git reflog
git reset --hard HEAD@{n}

# rebase 직후라면 더 간단하게
git reset --hard ORIG_HEAD
```

## 실무 적용

| 상황 | 명령 |
|------|------|
| feature를 최신 main 위로 | `git rebase main` |
| 커밋 정리 (squash 등) | `git rebase -i HEAD~n` |
| 브랜치 기반 변경 | `git rebase --onto new-base old-base branch` |
| 중간 커밋 제거 | `git rebase --onto A C branch` |
| fixup 커밋 자동 정리 | `git rebase -i --autosquash main` |
| dirty 상태에서 rebase | `git rebase --autostash main` |

### 주의사항

- **이미 push한 커밋은 rebase 지양** — 다른 사람의 히스토리가 깨짐
- rebase 후 원격 반영: `git push --force-with-lease` (`--force`보다 안전)
- merge vs rebase: **rebase**는 선형 히스토리, **merge**는 합류 지점 보존

## 출처 / 참고

- [Git 공식 문서 - git rebase](https://git-scm.com/docs/git-rebase)
- [[stacked-prs-rebase-onto|Git 스택 PR - rebase --onto로 정리하기]]

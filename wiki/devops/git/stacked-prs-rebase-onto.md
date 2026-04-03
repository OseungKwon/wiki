---
title: Git 스택 PR - rebase --onto로 정리하기
aliases:
  - stacked PRs
  - rebase onto
tags:
  - git
  - rebase
  - pull-request
  - workflow
created: 2026-03-18
updated: 2026-04-01
reviewed: false
---

## 정의

스택 PR(Stacked PRs)은 `main` ← `sub1` ← `sub2`처럼 브랜치를 순차적으로 쌓아 올려 작업하는 방식이다. `sub1`이 스쿼시 머지되면 `sub2`에 `sub1`의 커밋이 중복 포함되는 문제가 발생하며, `git rebase --onto`로 해결한다.

## 상세 설명

### 문제 상황

```
main ─── A ─── B
          \
           sub1 ─── C ─── D
                      \
                       sub2 ─── E ─── F
```

- `sub1` PR: 커밋 C, D (base: `main`) → 문제 없음
- `sub2` PR: 커밋 C, D, E, F (base: `sub1`) → `sub1`의 커밋이 모두 포함됨

`sub1`이 **스쿼시 머지**되면 상황이 더 꼬인다:

```
main ─── A ─── B ─── CD(squash)
          \
           sub1 ─── C ─── D       ← 원본 커밋 (삭제 예정)
                      \
                       sub2 ─── E ─── F
```

`sub2`에는 원본 C, D가 남아있고 main에는 스쿼시된 CD가 있어서 Git이 같은 변경으로 인식하지 못한다.

### 해결: `rebase --onto`

```bash
# sub1이 main에 스쿼시 머지된 후:
git checkout main && git pull
git checkout sub2
git rebase --onto main sub1 sub2
git push --force
```

**`rebase --onto main sub1 sub2`의 의미:**

```
rebase --onto <새 base> <기존 base> <대상 브랜치>
```

> "sub2에서 sub1 이후에 추가된 커밋(E, F)만 골라서 main 위에 얹어라"

**결과:**

```
main ─── A ─── B ─── CD(squash)
                        \
                         sub2 ─── E' ─── F'
```

sub2 PR에는 E, F 커밋만 깔끔하게 남는다.

### 전체 워크플로우

```
1. sub1 브랜치 생성 (base: main)
2. sub2 브랜치 생성 (base: sub1)
3. sub1 PR 생성 → base: main
4. sub2 PR 생성 → base: sub1  ← GitHub에서 base 설정
5. sub1 스쿼시 머지 완료
6. sub2 PR의 base를 main으로 변경 (GitHub UI)
7. git rebase --onto main sub1 sub2
8. git push --force
```

### 주의사항

- rebase 전에 **로컬에 `sub1` 브랜치가 남아있어야** 분기점을 정확히 지정 가능
- `sub1`이 이미 삭제됐다면:
  - `git reflog`에서 `sub1`의 마지막 커밋 해시를 찾아 사용
  - 또는 `git rebase -i main`으로 interactive rebase하여 수동 제거
- force push 전에 `sub2`에서 다른 사람이 작업하고 있지 않은지 확인

## 출처 / 참고
## 전제 조건: 작업 중 스쿼시 금지

이 가이드는 **sub1, sub2 모두 원본 커밋을 유지한 상태**에서 작업하는 것을 전제로 한다. 스쿼시는 GitHub 머지 시점에 GitHub이 처리하도록 맡기고, 로컬에서 미리 스쿼시하면 안 된다.

브랜치명을 old base로 사용하려면 해당 브랜치가 sub2의 **조상(ancestor)**이어야 하는데, 로컬 스쿼시를 하면 이 조상 관계가 깨진다.

### 로컬에서 이미 스쿼시한 경우

sub1을 로컬에서 스쿼시한 뒤 sub2를 딴 경우, sub1 브랜치가 sub2의 조상이 아니게 된다.

```
main ─── A ─── B
          \
           C ─── D (원본 개발 커밋)
           │       \
           │        sub2 ─── E ─── F
           │
           sub1 ─── CD(로컬 스쿼시)  ← sub2의 조상이 아님
```

이 경우 브랜치명 대신 **sub2 고유 커밋 직전의 커밋 해시**를 old base로 지정해야 한다:

```bash
git rebase --onto main <D의 커밋 해시> sub2
```

| | 정상 케이스 | 로컬 스쿼시 케이스 |
|--|--|--|
| sub1이 sub2 조상? | O | X |
| old base | `sub1` (브랜치명) | 커밋 해시 직접 지정 |
| 스쿼시 주체 | GitHub (머지 시) | 로컬 (머지 전) |

## 출처 / 참고

- [Git 공식 문서 - rebase --onto](https://git-scm.com/docs/git-rebase)

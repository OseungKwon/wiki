---
title: 브랜치를 다른 브랜치로 강제 덮어쓰기
aliases: [git reset hard branch, force overwrite branch, 브랜치 강제 동기화]
tags:
  - git
  - reset
  - force-push
  - workflow
created: 2026-06-19
updated: 2026-06-19
---

## 정의
브랜치 `a`를 브랜치 `b`의 내용과 히스토리째 완전히 동일하게 만드는 방법.
머지가 아니라 포인터를 옮기는 `reset --hard`라 충돌이 발생하지 않는다.

## 상세 설명
### 핵심 원리
`git merge`는 두 브랜치 내용을 합치므로 충돌이 날 수 있다. 반면
`git reset --hard b`는 `a`의 브랜치 포인터를 `b`가 가리키는 커밋으로 옮기고
작업트리를 통째로 그 상태에 맞춘다. "합치는" 게 아니라 "갈아끼우는" 것이라
충돌 개념 자체가 없다.

### 로컬 브랜치 덮어쓰기
```bash
git checkout a
git reset --hard b
```

### 원격에도 반영 (force push)
```bash
git push origin a --force-with-lease
```
> `--force-with-lease`는 `--force`보다 안전하다. 내가 마지막으로 본 이후
> 누군가 그 브랜치에 push한 게 있으면 거부해서, 남의 작업을 덮어쓰는 사고를 막아준다.

### 원격 브랜치 b 기준으로 맞추기
b가 원격에 있는 최신 브랜치라면 fetch부터 한다.
```bash
git fetch origin
git checkout a
git reset --hard origin/b
git push origin a --force-with-lease
```

## 트러블슈팅
- **`a`에만 있던 커밋이 사라진다**: `reset --hard`는 되돌릴 수 없게 작업트리를
  덮어쓴다. 보존이 필요하면 미리 `git branch backup-a`로 백업하거나
  `git reflog`로 복구한다.
- **force push가 거부됨(`--force-with-lease`)**: 원격이 내 로컬 기준보다
  앞서 있다는 뜻. `git fetch` 후 내용을 확인하고 다시 시도한다.

## 출처 / 참고
- [[Git Rebase 심화 가이드]]
- [[GitHub Merge 전략]]

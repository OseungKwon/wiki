---
title: GitHub 여러 계정 SSH 등록
aliases:
  - git 멀티 계정
  - github ssh 다중 계정
tags:
  - git
  - github
  - ssh
  - authentication
created: 2026-03-23
updated: 2026-03-23
---

## 정의
하나의 컴퓨터에서 여러 GitHub 계정(개인/회사 등)을 [[ssh|SSH]] 키로 구분하여 사용하는 방법.

## 상세 설명

### 설정 방법

#### 1. 계정별 SSH 키 생성

```bash
# 개인 계정
ssh-keygen -t ed25519 -C "personal@email.com" -f ~/.ssh/id_personal

# 회사 계정
ssh-keygen -t ed25519 -C "work@email.com" -f ~/.ssh/id_work
```

#### 2. `~/.ssh/config` 설정

Host별로 다른 키를 매핑한다.

```
# 개인 계정
Host github-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_personal

# 회사 계정
Host github-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_work
```

#### 3. GitHub에 공개키 등록

각 GitHub 계정의 Settings → SSH and GPG keys에 해당 `.pub` 파일 내용을 등록한다.

```bash
cat ~/.ssh/id_personal.pub   # 복사 후 개인 GitHub에 등록
cat ~/.ssh/id_work.pub       # 복사 후 회사 GitHub에 등록
```

#### 4. 리모트 URL 변경

`github.com` 대신 config에 설정한 Host명을 사용한다.

```bash
# 개인 프로젝트
git remote set-url origin git@github-personal:username/repo.git

# 회사 프로젝트
git remote set-url origin git@github-work:company/repo.git
```

#### 5. 디렉토리별 커밋 author 자동 분리

`~/.gitconfig`:
```gitconfig
[user]
    name = 개인이름
    email = personal@email.com

[includeIf "gitdir:~/Documents/work/"]
    path = ~/.gitconfig-work
```

`~/.gitconfig-work`:
```gitconfig
[user]
    name = 회사이름
    email = work@company.com
```

### 연결 테스트

```bash
ssh -T git@github-personal   # Hi personal! ...
ssh -T git@github-work        # Hi work! ...
```

## 출처 / 참고
- [GitHub 공식 - SSH 키 생성](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [[ssh|SSH 개념]]

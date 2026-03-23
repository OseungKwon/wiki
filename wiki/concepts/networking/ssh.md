---
title: SSH (Secure Shell)
aliases:
  - secure shell
tags:
  - networking
  - security
  - ssh
  - authentication
created: 2026-03-23
updated: 2026-03-23
---

## 정의
암호화된 네트워크 프로토콜로, 원격 컴퓨터에 안전하게 접속하기 위한 통신 방식.

## 상세 설명

### 핵심 원리: 비대칭 키 암호화

- **공개키 (Public Key)** — 자물쇠. 서버(GitHub 등)에 등록
- **개인키 (Private Key)** — 열쇠. 내 컴퓨터에만 보관

접속 시 서버가 공개키로 암호화한 질문을 보내면, 개인키로만 복호화할 수 있어 본인 인증이 된다.

### Git HTTPS vs SSH
### 기본 사용 흐름

```bash
# 1. 키 생성
ssh-keygen -t ed25519 -C "email@example.com"

# 2. 공개키를 서버에 등록
cat ~/.ssh/id_ed25519.pub  # 이 내용을 서버에 복사

# 3. 접속 (비밀번호 없이 인증)
ssh user@server.com
```

### 주요 용도

- **원격 서버 접속**: `ssh user@host`
- **Git 인증**: GitHub/GitLab 등에서 비밀번호 없이 push/pull
- **포트 포워딩/터널링**: 로컬과 원격 서버 간 안전한 통신 채널 구성
- **SCP/SFTP**: 암호화된 파일 전송

## 실무 적용
- Git에서 HTTPS 대신 SSH를 사용하면 토큰 만료 걱정 없이 인증 가능
- 여러 GitHub 계정 사용 시 SSH config로 Host별 키를 분리 → [[github-multiple-ssh-accounts|GitHub 여러 계정 SSH 등록]]

## 출처 / 참고
- [GitHub 공식 - SSH 키 생성](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)

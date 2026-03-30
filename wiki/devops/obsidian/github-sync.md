---
title: Obsidian GitHub 동기화
tags:
  - obsidian
  - github
  - git
  - sync
created: 2026-03-17
updated: 2026-03-30
---

## 정의
Obsidian Git 플러그인을 사용하여 GitHub 저장소를 통해 여러 기기 간 Obsidian Vault를 무료로 동기화하는 방법이다. Obsidian Sync(유료)의 대안.

## 개요
GitHub 저장소 생성, Obsidian Git 플러그인 설정, 다중 기기 동기화 방법을 다룬다.

## 배경
Obsidian은 로컬 파일 기반 마크다운 노트 앱으로, 유료 Obsidian Sync 없이도 Git을 활용하면 무료로 버전 관리와 다중 기기 동기화가 가능하다.

## 상세 설명

### 1. GitHub 저장소 생성
- GitHub에서 **Private 저장소** 생성 (노트 보안을 위해 비공개 필수)

### 2. 로컬 Vault를 Git 저장소로 초기화

```bash
cd ~/Documents/Obsidian\ Vault
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/<username>/<repo>.git
git branch -M main
git push -u origin main
```

### 3. Obsidian Git 플러그인 설치
- Obsidian 설정 → Community plugins → Browse → **"Obsidian Git"** 검색 → 설치 및 활성화

### 4. 플러그인 주요 설정

| 옵션 | 권장값 | 설명 |
|------|--------|------|
| Auto backup interval | 5~10분 | 자동 커밋+푸시 간격 |
| Pull on startup | ON | 앱 시작 시 최신 내용 풀 |
| Auto refresh | ON | 변경사항 자동 감지 |

### 5. .gitignore 설정

기기 간 충돌 방지를 위해 워크스페이스 관련 파일을 제외한다:

```
.obsidian/workspace.json
.obsidian/workspace-mobile.json
.trash/
.DS_Store
```

### 다중 기기 동기화

| 플랫폼 | 방법 |
|--------|------|
| PC/Mac | GitHub Desktop 또는 CLI로 클론 |
| iOS | Working Copy 앱으로 저장소 클론 |
| Android | Termux에서 git clone |


### 새 컴퓨터에서 연동 절차

**Step 1. GitHub 저장소 클론**
```bash
cd ~/Documents
git clone https://github.com/<username>/<repo>.git "Obsidian Vault"
```

**Step 2. Obsidian에서 Vault 열기**
- Obsidian 실행 → **Open folder as vault** → 클론한 폴더 선택

**Step 3. Obsidian Git 플러그인 설치**
- 설정 → Community plugins → Browse → **"Obsidian Git"** 검색 → 설치 및 활성화
- `.obsidian/plugins/` 설정이 동기화되어 있어도, 플러그인 자체는 새로 설치해야 활성화됨

**Step 4. 플러그인 설정 확인**
- 동기화된 설정이 자동 반영되므로 별도 설정 불필요
- Auto backup interval, Pull on startup 등이 기존과 동일한지 확인만 하면 됨

**주의사항**
- `.obsidian/workspace.json`은 `.gitignore`에 포함되어 기기별 충돌 없음
- **편집 전 항상 Pull 먼저** — 충돌 방지의 핵심

## 트러블슈팅
- **충돌 발생 시**: Pull on startup을 활성화하고, 편집 전 항상 최신 상태를 풀받는 습관 필요
- **`.obsidian/` 전체 제외 여부**: 플러그인 설정도 동기화하려면 `workspace.json`만 제외하고 나머지는 포함

## 출처 / 참고
- [Obsidian Sync 대안 (Git 플러그인) 25년도 버전](https://velog.io/@seukseok/Obsidian-Sync-%EB%8C%80%EC%95%88-Git-%ED%94%8C%EB%9F%AC%EA%B7%B8%EC%9D%B825%EB%85%84%EB%8F%84-%EB%B2%84%EC%A0%84)
- [Obsidian github 동기화 세팅하기](https://clarit7.github.io/obsidian_sync_setting/)
- [Obsidian git으로 컴퓨터-안드로이드 연동하기](https://velog.io/@rethinking21/Obsidian-git%EC%9C%BC%EB%A1%9C-%EC%BB%B4%ED%93%A8%ED%84%B0-%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-%EC%97%B0%EB%8F%99%ED%95%98%EA%B8%B0)
- 관련 문서: [[obsidian-mcp-claude-code]]

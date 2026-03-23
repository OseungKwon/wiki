---
title: Wiki Plugin for Claude Code
aliases: [wiki-plugin, 위키 플러그인]
tags:
  - claude-code
  - obsidian
  - mcp
  - plugin
  - wiki
created: 2026-03-17
updated: 2026-03-17
---

## 정의
Claude Code에서 대화 맥락, URL, 텍스트를 Obsidian Vault에 위키 문서로 정리하는 플러그인 (v2.1.0).

## 상세 설명

### 사전 조건
- Obsidian 설치 및 Vault 생성
- Obsidian MCP 서버 연결 설정
- Claude Code CLI

### 설정 방법

**루트 디렉토리**: 기본값 `wiki/`. 변경 방법 2가지:
- 명령 시 `root=<경로>` 옵션: `/wiki root=notes/tech Redis 정리`
- Vault 내 `wiki/_config.md`로 기본값 설정

**사용자 정의 템플릿**: `{ROOT}/_templates/`에 배치하면 기본 템플릿보다 우선 적용된다.

### 사용법

| 명령 | 모드 | 설명 |
|------|------|------|
| `/wiki <주제>` | A (대화 맥락) | 대화에서 해당 주제 추출하여 문서 생성 |
| `/wiki` + 긴 텍스트 | B (텍스트 정리) | 텍스트를 분류/구조화하여 저장 |
| `/wiki quick <주제>` | C (Quick) | 확인 단계 생략, 즉시 저장 |
| `/wiki update <경로>` | D (업데이트) | 기존 문서에 새 정보 추가 |
| `/wiki search <키워드>` | E (검색) | 카탈로그 + 전문 검색 |
| `/wiki list [필터]` | F (목록) | 문서 목록 조회 (태그/카테고리 필터) |
| `/wiki <URL>` | G (URL) | URL 콘텐츠를 위키 문서로 변환 |
| `/wiki help` | H (도움말) | 사용법 안내 |

### 핵심 구조

**카탈로그 (`_catalog.md`)**: 루트에 유지되는 문서 인덱스. 매번 디렉토리 탐색 대신 카탈로그만 읽어 기존 문서를 파악한다. `append_content`로만 업데이트.

**템플릿 3종**:
- **개념형 (A)**: 기술/이론 설명 — 정의, 상세 설명, 실무 적용
- **가이드형 (B)**: 설치/설정/사용법 — 사전 조건, 설정 방법, 사용법, 트러블슈팅
- **트러블슈팅형 (C)**: 에러 해결 — 증상, 원인, 해결

**자동 기능**:
- 관련 문서 `[[wikilink]]` 자동 연결 (태그 2개 이상 겹침 또는 제목 언급 시)
- MOC(Map of Content) 허브 문서 생성 (관련 문서 3개 이상 시)
- 문서 분리 판단 (독립적 주제 2개 이상, 200줄 초과 시)

### 아키텍처 (v2)

v2에서는 스킬 정의(`SKILL.md`)와 상세 절차(`references/`)가 분리되었다. 모드별로 필요한 절차 파일만 로드하여 토큰을 절약한다.

```
wiki-plugin/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── wiki/
│       ├── SKILL.md              # 모드 판별 + 실행 진입점
│       └── references/
│           ├── procedures.md     # 상세 실행 절차
│           └── templates.md      # 문서 템플릿
├── README.md
└── LICENSE
```

### 제약사항
- Obsidian MCP 도구만 사용 (로컬 Read/Write/Edit 사용 금지)
- 카탈로그는 `append_content`로만 업데이트 (`patch_content` 사용 금지)
- 검색/목록 모드(E, F)에서는 파일 수정 금지

## 출처 / 참고
- [[obsidian-mcp-claude-code|Obsidian MCP - Claude Code 연동]]

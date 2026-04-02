---
title: RTK (Rust Token Killer)
aliases:
  - rtk
  - rust-token-killer
tags:
  - claude-code
  - cli
  - token-optimization
  - rust
created: 2026-04-02
updated: 2026-04-02
---

## 정의

AI 코딩 에이전트(Claude Code 등)와 셸 명령어 사이에 위치하는 Rust 기반 CLI 프록시. 명령어 출력을 지능적으로 필터링/압축하여 LLM 토큰 소비를 60-90% 절감한다.

## 상세 설명

### Hook 기반 자동 리라이팅

`rtk init -g` 실행 시 Claude Code의 **PreToolUse 훅**으로 등록된다.

1. Claude Code가 Bash 도구로 명령어 실행 요청 (예: `git status`)
2. 훅 스크립트(`rtk-rewrite.sh`)가 가로채서 `rtk rewrite "git status"` 호출
3. exit code에 따라 분기:
   - **0**: 리라이트 성공 → `rtk git status`로 자동 교체 + auto-allow
   - **1**: 매칭 규칙 없음 → 원본 그대로 통과
   - **2**: deny 규칙 → 실행 차단
   - **3**: ask 규칙 → 리라이트하되 사용자 확인 요청

Claude는 `git status`를 쓰지만 실제로는 `rtk git status`가 실행된다. Claude에게 완전히 투명하게 동작한다.

### 6단계 명령어 라이프사이클

| 단계 | 역할 |
|------|------|
| PARSE | Clap 파서로 명령어/인자/글로벌 플래그 추출 |
| ROUTE | `Commands` enum으로 적절한 모듈 라우팅 |
| EXECUTE | `std::process::Command`로 실제 명령어 실행, stdout/stderr/exit code 캡처 |
| FILTER | 명령어별 필터링 전략 적용 |
| PRINT | 압축된 결과 출력 |
| TRACK | SQLite DB에 입출력 토큰 수/절감률 기록 |

### 12가지 필터링 전략

| 전략 | 예시 | 절감률 |
|------|------|--------|
| Stats Extraction | git status/log/diff → 통계 요약 | 90-99% |
| Failure Focus | 테스트 출력에서 실패만 추출 | 94-99% |
| Structure Only | JSON → 스키마만 (값 제거) | 80-95% |
| Progress Filtering | ANSI 프로그레스 바 제거, 최종 결과만 | 85-95% |
| Grouping by Pattern | lint 에러를 규칙/파일별 그룹핑 | 80-90% |
| Deduplication | 반복 로그 → 카운트로 축약 | 70-85% |
| Tree Compression | 파일 목록 → 디렉토리 트리 | 50-70% |
| Error Only | stderr만, stdout 제거 | 60-80% |
| Code Filtering | 주석/함수 본문 제거 (aggressive 모드) | 20-90% |
| JSON/Text Dual Mode | JSON output 지원 도구에서 JSON 사용 | 80%+ |
| State Machine Parsing | 테스트 상태 추적 파싱 | 90%+ |
| NDJSON Streaming | 라인별 JSON 이벤트 집계 | 90%+ |

### 아키텍처

```
src/
├── cmds/          # 42개 명령어 모듈 (git/, js/, python/, go/, rust/ 등)
├── core/          # filter.rs, config.rs, tracking.rs(SQLite), tee.rs
├── hooks/         # init.rs(설치), rewrite_cmd.rs(리라이트), permissions.rs
├── discover/      # rules.rs(70+개 정규식), registry.rs(RegexSet 매칭)
└── analytics/     # gain.rs(절감 대시보드), session_cmd.rs
```

- `discover/rules.rs`: 70+개 정규식 패턴으로 명령어 → RTK 매핑 테이블 관리
- `discover/registry.rs`: 컴파일된 `RegexSet`으로 O(n) 대신 O(1)에 가까운 빠른 매칭
- `core/tee.rs`: 필터링 실패 시 원본 출력 복구 (fail-safe)
- `core/tracking.rs`: SQLite(`~/.local/share/rtk/history.db`)에 토큰 절감 이력 저장

### 안전 장치

- **Exit code 보존**: 원본 명령어의 exit code를 그대로 반환하여 CI/CD 파이프라인에서 안전
- **Fail-safe**: 필터링 실패 시 원본 출력 그대로 반환
- **Tee 시스템**: 전체 미필터 출력을 저장, 실패 시 `rtk proxy <cmd>`로 원본 확인 가능
- **프록시 오버헤드**: ~5-15ms로 체감 불가

### 설정 파일

| 파일 | 용도 |
|------|------|
| `~/.config/rtk/config.toml` | 글로벌 설정 (DB 경로, 제외 명령어, tee 동작) |
| `.rtk/filters.toml` | 프로젝트별 커스텀 필터 |
| `~/.claude/RTK.md` | Claude에게 rtk 사용법 안내 |
| `~/.claude/settings.json` | PreToolUse 훅 등록 |

### 메타 명령어

```bash
rtk gain              # 토큰 절감 분석 대시보드
rtk gain --history    # 명령어별 사용 이력과 절감률
rtk discover          # Claude Code 히스토리 분석하여 놓친 최적화 기회 탐색
rtk proxy <cmd>       # 필터링 없이 원본 명령어 실행 (디버깅용)
```

## 출처 / 참고

- GitHub: https://github.com/rtk-ai/rtk
- 관련 문서: [[Claude Code Hooks]]

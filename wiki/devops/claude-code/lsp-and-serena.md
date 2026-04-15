---
title: LSP와 Serena - AI 코드 탐색 도구 비교
aliases: [Language Server Protocol, LSP vs Serena, 타입 정의 탐색]
tags:
  - lsp
  - serena
  - claude-code
  - mcp
  - code-intelligence
created: 2026-04-15
updated: 2026-04-15
reviewed: false
---

## 정의

LSP(Language Server Protocol)는 에디터/AI와 언어별 분석 서버 사이의 표준 통신 프로토콜이다.
Serena는 LSP를 래핑 + 확장한 MCP 서버로, Claude Code에서 코드베이스를 탐색할 때 사용한다.

## 상세 설명

### LSP 동작 원리

```
[에디터/Claude] ←── JSON-RPC ──→ [Language Server]
                                        ↓
                              실제 컴파일러/타입체커 실행
                              (TypeScript: tsserver)
```

에디터가 언어별 분석을 직접 하지 않고 **언어 서버에 질의**하는 구조.
MS가 2016년 VS Code용으로 만들었으며 이후 표준화됨.

**동작 흐름:**
1. 언어 서버가 백그라운드에서 프로젝트 전체를 인덱싱
2. `tsconfig.json` 기준으로 타입 그래프 구성
3. 질의가 오면 실제 TypeScript 컴파일러 API로 응답

Grep과 달리 **의미론적(semantic)** 이해 — 같은 이름 변수라도 스코프가 다르면 다른 심볼로 인식.

### LSP 주요 오퍼레이션

| 오퍼레이션 | 설명 |
|-----------|------|
| `goToDefinition` | 심볼 정의 위치로 이동 |
| `findReferences` | 심볼의 모든 참조 찾기 |
| `hover` | 타입 추론 결과 확인 |
| `documentSymbol` | 파일 내 전체 심볼 목록 |
| `workspaceSymbol` | 워크스페이스 전체 심볼 검색 |
| `incomingCalls` / `outgoingCalls` | 호출 계층 분석 |

### AI 코드 탐색 방식 비교

기존 방식(Grep/Bash)과 LSP의 차이:

| | Grep/find | LSP |
|---|---|---|
| 결과 | 파일 경로만 | 파일 + 정확한 라인 번호 |
| 추가 작업 | 각 파일 Read → 위치 재탐색 | 즉시 해당 라인으로 점프 |
| 누락 위험 | 패턴 틀리면 miss | 심볼 기반, 100% 정확 |
| 컨텍스트 소모 | 파일 여러 개 전체 Read | 필요한 라인만 Read |
| 스코프 구분 | 불가 | 정확한 심볼 단위 |

**실제 효과**: 타입 정의 확인 시 2~3단계 탐색이 1단계로 줄어든다.

### LSP vs Serena

| | LSP (Claude Code 내장) | Serena MCP |
|---|---|---|
| 기반 | tsserver / pylsp 등 언어별 서버 | LSP + 자체 추가 분석 |
| 주요 기능 | goToDefinition, findReferences, hover, documentSymbol | + 심볼 검색, 파일 개요, 코드 청크 추출 |
| 반환 정보 | 위치 (파일:라인:컬럼) | 위치 + **실제 코드 내용** |
| 컨텍스트 소모 | 위치만 → 별도 Read 필요 | 코드 내용 포함 → Read 생략 가능 |
| 속도 | 빠름 (이미 인덱싱됨) | 초기 인덱싱 시간 있음 |
| 정확도 | 컴파일러 수준 | 동일 (LSP 기반) |

Serena는 LSP를 **래핑 + 확장**한 구조다.

### 언제 뭘 쓰나

- **LSP**: 타입 에러 원인 추적, 참조 전체 파악, enum 멤버 목록 확인, 세밀한 참조 분석
- **Serena**: 심볼 정의 내용 바로 보기, 파일 구조 파악, 코드 청크 단위 탐색
- **Serena가 없을 때**: LSP가 보완재

## 실무 적용

`findReferences`로 enum 참조 43개를 11개 파일에서 정확한 라인까지 한 번에 확인.
`documentSymbol`로 `QueryKeys` enum 전체 멤버를 Read 없이 즉시 파악.

## 출처 / 참고

- [[mcp-tool-token-consumption]] — MCP 도구 토큰 소모 구조

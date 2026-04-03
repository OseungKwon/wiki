---
title: MCP 도구 호출과 토큰 소모 구조
aliases:
  - MCP token consumption
  - tool_result tokens
tags:
  - mcp
  - token
  - optimization
  - claude-code
created: 2026-03-19
updated: 2026-03-19
reviewed: false
---

## 정의
MCP 도구 호출 시 Claude의 토큰 소모는 MCP 서버의 외부 API 통신이 아니라, Claude와 주고받는 메시지(tool_use, tool_result)의 크기에 의해 결정된다.

## 상세 설명

### 토큰 소모 흐름

```
사용자 메시지                    → input tokens
Claude가 도구 호출 결정 (tool_use) → output tokens (적음)
MCP 서버 ↔ 외부 API 통신          → 토큰 무관
tool_result가 Claude에 반환       → input tokens (가장 큼)
Claude의 최종 응답 생성            → output tokens
```

핵심: **tool_result로 Claude에게 전달되는 내용의 크기**가 input tokens를 결정한다.

### Hook을 통한 토큰 절약

MCP → Hook → Claude 구조에서 Hook이 MCP 결과를 가공하여 축소하면, Claude에게 전달되는 tool_result가 작아지므로 input token 소모가 줄어든다.

예: Figma `get_design_context`가 대량 데이터를 반환해도, Hook이 필요한 필드만 추출하면 토큰 소모를 대폭 줄일 수 있다.

### Claude Code의 MCP 출력 제한

- MCP 도구 출력이 **10,000 토큰 초과 시 경고** 표시
- `MAX_MCP_OUTPUT_TOKENS` 환경변수로 제한 조절 가능 (예: `MAX_MCP_OUTPUT_TOKENS=50000`)
- Tool Search 기능: 도구 정의가 컨텍스트의 10% 이상을 차지하면 자동으로 on-demand 로딩 전환 (최대 85% 컨텍스트 절약)

### 주의사항

- Hook이 결과를 **과도하게 축소**하면 Claude가 충분한 정보를 받지 못해 부정확한 응답을 생성할 수 있다
- 필요한 정보는 유지하면서 불필요한 메타데이터, 중복 데이터만 제거하는 것이 핵심

## 출처 / 참고
- [Token counting - Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/token-counting)
- [Optimising MCP Server Context Usage in Claude Code](https://scottspence.com/posts/optimising-mcp-server-context-usage-in-claude-code)
- [Connect Claude Code to tools via MCP](https://code.claude.com/docs/en/mcp)
- [[Claude Code Hooks]]
- [[Obsidian MCP - Claude Code 연동]]

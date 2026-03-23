---
title: Claude Code Hooks
aliases:
  - Claude Hook
  - PostToolUse Hook
tags:
  - claude-code
  - hook
created: 2026-03-19
updated: 2026-03-19
---

## 정의

Claude Code가 특정 이벤트 발생 시 외부 셸 명령을 자동 실행하는 메커니즘. 도구 호출 전후, 알림 시점 등에 사용자 정의 스크립트를 끼워 넣어 동작을 확장할 수 있다.

## 상세 설명

### 설정 위치

`~/.claude/settings.json`의 `hooks` 필드에 등록한다:

```json
{
  "hooks": {
    "<이벤트명>": [
      {
        "matcher": "<매칭 조건>",
        "hooks": [
          {
            "type": "command",
            "command": "<실행할 셸 명령>"
          }
        ]
      }
    ]
  }
}
```

### Hook 이벤트 종류

| 이벤트 | 시점 | stdin 입력 | stdout 반환 |
|--------|------|-----------|-------------|
| `PreToolUse` | 도구 실행 **직전** | tool_name, tool_input | 실행 차단 가능 |
| `PostToolUse` | 도구 실행 **직후** | tool_name, tool_input, tool_response | 응답 교체 가능 |
| `Notification` | 사용자 입력 대기 등 알림 시 | 알림 내용 | — |

### matcher

어떤 도구/이벤트에 반응할지 결정하는 문자열 매칭 조건.

```json
"matcher": "mcp__claude_ai_Figma__get_design_context"
"matcher": "Read"
"matcher": "Bash"
"matcher": "user-prompt-submit"
```

### 데이터 흐름 (PostToolUse 기준)

```
Claude Code가 도구 호출 → 응답 수신
                             │
                     matcher 문자열 매칭
                             │
               스크립트 실행 (stdin으로 JSON 전달)
               ┌─────────────┴─────────────┐
               │ {                          │
               │   tool_name: string,       │
               │   tool_input: object,      │
               │   tool_response: object    │  ← MCP 원본 응답
               │ }                          │
               └─────────────┬─────────────┘
                             │
                     스크립트가 변환 수행
                             │
               ┌─────────────┴─────────────┐
               │ {                          │
               │   hookSpecificOutput: {    │
               │     hookEventName:         │
               │       "PostToolUse",       │
               │     updatedMCPToolOutput:  │  ← 교체할 응답
               │       [{ type, text }]     │
               │   }                        │
               │ }                          │
               └───────────────────────────┘
                             │
               Claude는 교체된 응답만 본다
```

- `{}` (빈 객체) 반환 시 원본 응답 그대로 사용
- `updatedMCPToolOutput` 반환 시 원본 대신 교체

### 활용 예시

**응답 변환** — MCP 도구 응답에서 불필요한 부분을 제거하거나 정보를 추가

**알림** — 사용자 입력 대기 시 macOS 알림 표시:
```json
{
  "matcher": "user-prompt-submit",
  "hooks": [{
    "type": "command",
    "command": "osascript -e 'display notification \"입력 대기\" with title \"Claude Code\"'"
  }]
}
```

**실행 차단** — PreToolUse에서 특정 도구 호출을 조건부로 막기

## 출처 / 참고

- Claude Code 공식 문서: Hooks

---
title: Obsidian MCP - Claude Code 연동
tags:
  - mcp
  - obsidian
  - claude-code
  - automation
created: 2026-03-17
updated: 2026-03-17
reviewed: false
---

## 정의
Obsidian MCP는 Model Context Protocol을 통해 AI 도구(Claude Desktop, Claude Code 등)에서 Obsidian Vault의 파일을 직접 읽기/쓰기/검색할 수 있게 하는 서버이다.

## 개요
Obsidian MCP를 Claude Code CLI에 연동하는 방법과 Claude Desktop과의 설정 차이를 다룬다.

## 배경
AI 코딩 도구에서 Obsidian 노트를 직접 조작할 수 있으면 위키 자동화, TIL 기록, 프로젝트 문서화 등을 대화 흐름 안에서 처리할 수 있다.

## 상세 설명

### 사전 조건
1. **Obsidian** 실행 중이어야 함
2. **Local REST API** 커뮤니티 플러그인 설치 및 활성화
3. 플러그인 설정에서 **API Key** 복사
4. **uv** 패키지 매니저 설치 (`uvx` 명령 필요)

### Claude Desktop 설정
설정 파일 경로:
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "mcp-obsidian": {
      "command": "uvx",
      "args": ["mcp-obsidian"],
      "env": {
        "OBSIDIAN_API_KEY": "<your_api_key>"
      }
    }
  }
}
```

### Claude Code (CLI) 설정
Claude Code는 별도의 설정 파일을 사용한다:
- 글로벌: `~/.claude/settings.json`
- 프로젝트별: `.claude/settings.json`

```json
{
  "mcpServers": {
    "mcp-obsidian": {
      "command": "uvx",
      "args": ["mcp-obsidian"],
      "env": {
        "OBSIDIAN_API_KEY": "<your_api_key>"
      }
    }
  }
}
```

### 사용 가능한 MCP 도구
| 도구 | 용도 |
|------|------|
| `obsidian_get_file_contents` | 파일 읽기 |
| `obsidian_append_content` | 파일 생성/내용 추가 |
| `obsidian_patch_content` | 특정 섹션 수정 |
| `obsidian_simple_search` | 텍스트 검색 |
| `obsidian_list_files_in_dir` | 디렉토리 조회 |
| `obsidian_delete_file` | 파일 삭제 |

### 트러블슈팅
- `uvx`를 찾을 수 없는 경우: `which uvx`로 전체 경로를 확인하여 `command`에 입력
- Obsidian이 실행 중이지 않으면 MCP 연결 실패
- Local REST API 플러그인이 비활성화되어 있으면 404 에러 발생

## 출처 / 참고
- [옵시디언 MCP + Claude 사용해 기록 자동화하기 (velog)](https://velog.io/@ikseong00/MCP-%EC%98%B5%EC%8B%9C%EB%94%94%EC%96%B8-MCP-Claude-%EC%82%AC%EC%9A%A9%ED%95%B4-%EA%B8%B0%EB%A1%9D-%EC%9E%90%EB%8F%99%ED%99%94%ED%95%98%EA%B8%B0)
- [mcp-obsidian GitHub](https://github.com/MarkusPfworlds/mcp-obsidian)
- [[wiki-plugin|Wiki Plugin for Claude Code]]

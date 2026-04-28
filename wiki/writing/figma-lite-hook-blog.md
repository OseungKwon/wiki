---
title: "Claude Code에서 Figma 응답을 줄이는 hook을 만들었다"
aliases:
  - figma-lite-hook
  - figma hook 블로그
tags:
  - writing
  - blog
  - claude-code
  - figma
  - mcp
  - hook
  - token-optimization
created: 2026-04-28
updated: 2026-04-28
reviewed: false
---

# Claude Code에서 Figma 응답을 줄이는 hook을 만들었다

[Figma MCP](https://www.figma.com/community/app/1578169397428523117/figma-mcp-in-claude-code)의 `get_design_context`로 디자인을 코드로 변환할 수 있다. 그런데 이 응답이 크다. 복잡한 화면 하나 요청하면 수만 토큰이 나온다. StatusBar, Chrome toolbar 같은 디바이스 프레임, 반복되는 테이블 행, 쓰이지 않는 서브 컴포넌트 함수가 전부 딸려온다.

Claude Code에는 [PostToolUse hook](https://docs.anthropic.com/en/docs/claude-code/hooks)이라는 게 있다. 도구 응답이 Claude에게 전달되기 직전에 가로채서 바꿀 수 있는 구조다. 이걸로 Figma 응답을 정리하는 hook을 만들어서 쓰고 있는데, 어떻게 돌아가는지 정리해본다.

> 관련 위키: [[Claude Code Hooks]], [[MCP 도구 호출과 토큰 소모 구조]]

## 구조

파일이 두 개다. Shell 스크립트 하나, JS 번들 하나.

### Shell wrapper: `post-figma-design.sh`

Claude Code가 `get_design_context`를 호출하면 `settings.json`의 PostToolUse matcher가 shell 스크립트를 실행한다. hook 설정 방법은 [Hooks reference](https://docs.anthropic.com/en/docs/claude-code/hooks)에 나와 있다.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "mcp__claude_ai_Figma__get_design_context",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/post-figma-design.sh"
          }
        ]
      }
    ]
  }
}
```

이 스크립트가 하는 일:

1. `.claude/.env`에서 `FIGMA_ACCESS_TOKEN`을 로드
2. 응답이 파일로 overflow된 경우를 감지해서 hook이 읽을 수 있는 형식으로 변환
3. JSON을 `figma-lite-hook.js`에 stdin으로 넘김

2번이 좀 까다로운 부분이다. Figma 응답이 일정 크기를 넘으면 Claude Code가 내용을 파일로 저장하고 경로만 넘긴다. Shell 스크립트가 이걸 잡아서 `Result saved to file: <path>` 형식으로 바꿔주면, JS 쪽에서 해당 파일을 직접 읽는다.

### Transform engine: `figma-lite-hook.js`

[Bun](https://bun.sh/)으로 돌리는 JS 번들이다. stdin에서 JSON을 받아서 `tool_name`이 `get_design_context`이면 변환 파이프라인을 돌린다.

## 변환 파이프라인

### Figma API로 인스턴스 정보 보충

`get_design_context` 응답에 컴포넌트 인스턴스의 variant 정보가 빠져 있다. 그래서 hook이 [Figma REST API](https://www.figma.com/developers/api)를 따로 호출한다.

```
GET https://api.figma.com/v1/files/{fileKey}/nodes?ids={nodeId}
```

이 [GET file nodes](https://www.figma.com/developers/api#get-file-nodes-endpoint) 엔드포인트에서 `INSTANCE` 타입 노드를 재귀로 수집하고, `componentProperties`(variant, text props)와 `overrides`를 뽑아낸다. 이걸 `data-variant`, `data-text-props`, `data-overrides` 속성으로 JSX에 붙인다.

API가 실패하면 heuristic 모드로 간다. 코드에 이미 있는 `data-name`에서 variant를 추론한다. 타임아웃은 30초.

### 노이즈 제거

서브 컴포넌트 함수부터. Figma가 생성한 코드에는 메인 외에 `function Button()`, `function Card()` 같은 서브 컴포넌트가 여러 개 딸려온다. 이 함수들을 Props 타입 정의와 함께 삭제한다. 삭제 전에 함수 안에 있던 variant 정보를 빼두고, 메인 JSX의 해당 태그에 `data-variant`로 옮긴다.

다음으로 data-name 승격이 있다. Figma 레이어 이름이 `data-name` 속성으로 들어오는데, 이 중 실제 Figma 인스턴스와 이름이 같은 것들을 찾아서 `<div data-name="search-input">`을 `<SearchInput>`으로 바꾼다. Claude가 컴포넌트 구조를 파악하기 쉬워진다.

`ignore-components.json`에 무시 목록도 있다:

- `ignoreByDataName`: StatusBar, Chrome toolbar, Android Navigation Bar 같은 디바이스 프레임을 data-name으로 찾아서 JSX 블록째 날림
- `ignoreComponents`: PascalCase 태그명으로 매칭되는 self-closing 태그 삭제 (예: `<StatusBar ... />`)
- `stripChildren`: `<Icon>` 같은 태그의 children만 삭제하고 self-closing으로 변환

`const imgXxx = "data:image/..."` 형태의 base64 상수 중 JSX에서 안 쓰이는 것도 삭제한다.

### 압축

"These styles are contained in the design:" 같은 Figma 내부 메타데이터 블록을 날린다.

같은 태그가 3번 이상 나오면 공통 Tailwind 클래스를 뽑아서 `/* Component base styles */` 주석으로 빼고, 각 태그에는 고유 클래스만 남긴다.

구조가 같은 `<Table>` 블록이 여러 개면 첫 번째만 남기고 `{/* x N 동일 구조 (총 M행) */}` 주석으로 바꾼다. 비교할 때 `data-node-id`나 텍스트 내용은 정규화해서 무시한다.

`<div>`, `<span>` 같은 HTML 기본 태그에서 `data-node-id`를 뺀다. PascalCase 컴포넌트의 node ID는 남겨둔다.

마지막으로 남은 `data-name` 속성 제거, className 공백 정리, 연속 빈 줄 정리.

## 결과

복잡한 화면에서 50~70% 정도 토큰이 줄어든다. 단순한 컴포넌트는 별 차이 없고, 테이블이 들어간 대시보드 화면에서 효과가 크다.

## 왜 이렇게 만들었나

Shell과 JS를 분리한 건, 환경변수 로딩이나 overflow 감지는 shell이 편하고 JSX 정규식 치환은 JS가 편해서다. 하나로 합칠 이유가 없었다.

AST 파서를 안 쓰는 이유도 단순하다. Figma가 생성하는 코드는 구조가 예측 가능해서, 정규식 + 수동 bracket matching으로 처리된다. Bun에서 돌리면 수십 ms. AST 파서를 넣으면 의존성이 늘고 느려지는데, 그만큼의 정확도 이득이 없었다.

정보를 빼면서 동시에 추가하는 게 좀 이상해 보일 수 있다. Figma MCP 응답에는 Claude한테 필요 없는 것(디바이스 프레임, 메타데이터)과 빠져 있는 것(variant, overrides)이 같이 섞여 있다. 쓸모없는 걸 빼고 빠진 걸 채우는 양쪽을 다 해야 한다.

## hook 시스템에 대해서

Claude Code의 PostToolUse hook은 도구 호출 결과가 Claude에게 넘어가기 전에 끼어들 수 있는 구조다. Figma 말고도 API 응답 필터링이나 코드 생성 후처리 같은 데 쓸 수 있다.

컨텍스트 윈도우에 뭘 넣느냐가 생성 결과에 영향을 준다. 토큰을 줄이는 건 비용도 비용이지만, Claude가 읽어야 할 정보에서 쓸모없는 부분을 빼는 게 더 큰 이유다.

## 출처 / 참고

- [Claude Code Hooks reference](https://docs.anthropic.com/en/docs/claude-code/hooks) - PostToolUse를 포함한 hook 시스템 전체 문서
- [Figma REST API](https://www.figma.com/developers/api) - GET file nodes 등 엔드포인트 레퍼런스
- [Figma MCP server guide](https://github.com/figma/mcp-server-guide) - Figma MCP 서버 사용 가이드
- [Figma MCP in Claude Code](https://www.figma.com/community/app/1578169397428523117/figma-mcp-in-claude-code) - Claude Code용 Figma MCP 커뮤니티 앱
- [Bun](https://bun.sh/) - hook 실행에 사용한 JS 런타임

---
title: Claude 4.6 프롬프트 최적화 기법
aliases:
  - Claude 프롬프트 최적화
  - Claude prompting optimization
tags:
  - claude
  - prompt-engineering
  - llm
  - optimization
created: 2026-03-17
updated: 2026-03-17
reviewed: false
---

## 정의
Claude 4.6 모델에서 출력 품질, 일관성, 효율을 높이기 위한 7가지 핵심 프롬프트 최적화 기법을 정리한 문서이다.

## 상세 설명

### 1. XML 태그 구조화
프롬프트의 각 요소(지시, 컨텍스트, 예시)를 `<instructions>`, `<example>`, `<formatting>` 등 서술적 태그로 분리한다. Claude가 모호함 없이 파싱하여 복합 프롬프트에서 출력 품질이 크게 향상된다. 중첩 태그도 지원하여 계층적 콘텐츠 표현이 가능하다.

### 2. Few-shot 예시 (3-5개)
설명으로 전달하기 어려운 미묘한 요구사항(네이밍 컨벤션, 코드 스타일 등)을 예시가 직접 보여준다 (show > tell). Claude 4.x는 예시의 세부 디테일까지 복제하므로 원하는 행동과 정확히 일치하는 예시 제공이 중요하다.

좋은 예시의 3가지 기준:
- **관련성** — 실제 사용 사례와 유사
- **다양성** — 엣지 케이스를 커버
- **구조화** — `<example>` 태그로 감싸서 지시사항과 구분

### 3. 긍정문 우선
"~하지 마라"보다 "~해라"가 더 정확한 출력을 유도한다. LLM은 대화의 시작 방향을 이어가는 특성이 있어, 부정문은 금지된 행동을 오히려 활성화(prime)할 수 있다.

예: "마크다운을 사용하지 마라" → "plain text로 출력한다"

### 4. 공격적 지시어 완화
Opus 4.5/4.6는 시스템 프롬프트에 훨씬 민감하게 반응한다. 과거 모델의 undertriggering을 방지하려 넣었던 강한 표현이 overtriggering을 유발한다.

| 변환 전 | 변환 후 |
|---------|---------| 
| `CRITICAL: You MUST use this tool` | `Use this tool when...` |
| `ALWAYS do X` | `Do X` |
| `NEVER skip...` | `Skip하지 않도록 한다` |
| `If in doubt, use [tool]` | 제거 (4.6에서 적정 트리거됨) |

### 5. 적응형 사고 (Adaptive Thinking)
`thinking: {type: "adaptive"}`를 사용하면 Claude가 작업 복잡도에 따라 사고 깊이를 자동 조절한다. effort 파라미터(`low`/`medium`/`high`/`max`)로 토큰 소비와 응답 품질 간 트레이드오프를 제어한다.

| effort | 적합한 상황 |
|--------|-------------|
| low | 고볼륨, 지연시간 민감, 채팅 |
| medium | 에이전트 코딩, 도구 사용, 코드 생성 |
| high (기본) | 복잡한 추론, 분석 |
| max | 최고 품질이 필요한 경우 |

### 6. 긴 문서 상단 배치
20K+ 토큰의 긴 문서를 프롬프트 상단(질문/지시보다 위)에 배치하면 성능이 최대 30% 향상된다. "Lost in the Middle" 현상 — LLM은 컨텍스트의 맨 앞과 맨 끝에 있는 정보를 가장 잘 활용하고, 중간에 위치한 정보는 성능이 저하된다.

### 7. 병렬 도구 호출
독립적인 도구 호출을 동시에 실행하여 지연시간과 토큰을 절감한다. 도구를 stateless하고 parallelizable하게 설계하면 Claude가 자동으로 병렬성을 활용한다.

## 출처 / 참고
- [XML tags — Anthropic Docs](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-xml-tags)
- [Multishot prompting — Anthropic Docs](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/multishot-prompting)
- [Claude 4 best practices — Anthropic Docs](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/claude-4-best-practices)
- [Long context tips — Anthropic Docs](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/long-context-tips)
- [Adaptive thinking — Anthropic Docs](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking)
- [Programmatic tool calling — Anthropic Docs](https://platform.claude.com/docs/en/agents-and-tools/tool-use/programmatic-tool-calling)
- 관련 문서: [[claude-prompting-best-practices]]

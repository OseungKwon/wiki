---
title: Claude 프롬프팅 모범 사례 (2026.02 기준)
aliases:
  - Claude prompting best practices
  - Claude 프롬프팅 가이드
tags:
  - claude
  - prompt-engineering
  - llm
  - opus-4-6
  - migration
created: 2026-03-17
updated: 2026-03-17
reviewed: false
---

## 정의
Claude Opus 4.6, Sonnet 4.6, Haiku 4.5 모델 기준의 프롬프팅 종합 가이드이다. 일반 원칙, 출력 제어, 도구 사용, 에이전틱 시스템, Opus 4.6 마이그레이션까지 포함한다.

## 상세 설명

### 일반 원칙

**명확하고 직접적으로 지시**: 작성한 프롬프트를 맥락이 없는 동료에게 보여주고 따라해 보라고 해라. 그 동료가 혼란스러워하면 Claude도 혼란스러워할 것이다.

**컨텍스트(이유) 제공**: 지시의 이유나 동기를 함께 설명하면 Claude가 목표를 더 잘 이해한다. 예: `NEVER use ellipses` → `Your response will be read aloud by a text-to-speech engine, so never use ellipses since the engine will not know how to pronounce them.`

**역할 부여**: 시스템 프롬프트에서 역할을 설정하면 행동과 톤이 해당 용도에 집중된다.

### 출력 및 포맷팅

Claude 최신 모델은 더 직접적이고, 대화적이고, 간결하다. 도구 호출 후 요약이 필요하면 명시적으로 요청한다.

응답 형식 제어 4가지 방법:
1. "~하지 마라" 대신 원하는 형식을 직접 지시
2. XML 형식 지시자 활용 (`<smoothly_flowing_prose_paragraphs>`)
3. 프롬프트 스타일을 원하는 출력 스타일에 맞추기
4. Opus 4.6는 수학 표현에 LaTeX를 기본 사용 — 원하지 않으면 명시적으로 plain text 요청

**Prefill 제거 (Breaking Change)**: Claude 4.6부터 마지막 assistant 턴의 prefill 미지원. Structured Outputs 또는 시스템 프롬프트 지시로 대체.

### 도구 사용

**행동을 원하면 명확히 지시**: `Can you suggest some changes?` → `Change this function to improve its performance.`

**시스템 프롬프트 민감도 증가**: Opus 4.5/4.6는 이전 모델보다 훨씬 더 민감하게 반응. 공격적 프롬프트가 overtriggering 유발 → `"CRITICAL: You MUST use..."` → `"Use this tool when..."` 으로 완화.

**병렬 도구 호출**: 최신 모델은 프롬프팅 없이도 높은 성공률이나, ~100%까지 높이려면 `<use_parallel_tool_calls>` 태그로 명시.

### 사고 및 추론

**Opus 4.6 과도한 탐색**: 이전 모델보다 훨씬 많은 사전 탐색을 수행한다. 포괄적 기본값 대신 타겟 지시로 교체. `"Default to using [tool]"` → `"Use [tool] when it would enhance your understanding"`

**Adaptive Thinking**: `thinking: {type: "adaptive"}` 권장. effort 파라미터와 쿼리 복잡도에 기반하여 사고량을 동적 조절.

**사고 팁**: 일반 지시(`"think thoroughly"`)가 수동 단계별 계획보다 나은 추론 결과. 자기 검증 요청도 효과적.

### 에이전틱 시스템

**자율성과 안전성 균형**: Opus 4.6는 지침 없이 파일 삭제, force-push 등 위험한 행동을 취할 수 있다. 되돌리기 어려운 행동은 사용자 확인을 받도록 명시.

**서브에이전트 과다 생성**: Opus 4.6는 grep 한 번이면 충분한 작업에도 서브에이전트를 생성할 수 있다. 병렬/격리가 필요한 경우에만 서브에이전트를, 단순 작업은 직접 수행하도록 지시.

**과도한 엔지니어링 방지**: 요청하지 않은 추상화, 추가 파일, 유연성 추가 경향이 있다. 직접 요청된 변경만 수행하도록 명시.

### Opus 4.6 주요 변경사항

| 영역 | 이전 모델 | Opus 4.6 |
|------|----------|----------|
| 커뮤니케이션 | 장황, 자기 자축적 | 간결하고 직접적 |
| 시스템 프롬프트 민감도 | 공격적 프롬프팅 필요 | 과민 반응 가능 → 완화 필요 |
| 도구 triggering | Undertrigger 경향 | 적절하거나 overtrigger 경향 |
| 사전 탐색 | 제한적 | 매우 적극적 탐색 |
| 서브에이전트 사용 | 지시 시에만 | 자발적으로 과다 생성 경향 |
| 엔지니어링 수준 | 적절 | 과도한 추상화/기능 추가 경향 |
| Prefill | 지원 | 미지원 (Breaking Change) |
| 수학 표현 | 텍스트 혼용 | LaTeX 기본값 |

### 신규 기능

| 기능 | 설명 |
|------|------|
| Adaptive Thinking | `thinking: {type: "adaptive"}` - 사고량 동적 결정 |
| Effort 파라미터 GA | 베타 헤더 불필요. `max` 레벨 추가 |
| 128K 출력 토큰 | 이전 64K의 두 배 |
| Fast Mode | `speed: "fast"` - 최대 2.5배 빠른 출력 |
| Compaction API | 서버 측 자동 컨텍스트 요약 |

## 출처 / 참고
- [Prompting Best Practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)
- [What's New in Claude 4.6](https://platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-6)
- [Migration Guide](https://platform.claude.com/docs/en/about-claude/models/migration-guide)
- [Adaptive Thinking](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking)
- [Effort Parameter](https://platform.claude.com/docs/en/build-with-claude/effort)
- 관련 문서: [[claude-prompt-optimization]]

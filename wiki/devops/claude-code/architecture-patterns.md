---
title: Claude Code 시스템과 소프트웨어 아키텍처 패턴
aliases:
  - Claude Code 아키텍처
  - CLAUDE.md 분리 전략
tags:
  - claude-code
  - architecture
  - design-pattern
  - context-engineering
created: 2026-03-30
updated: 2026-03-30
---

## 정의

Claude Code의 구성 시스템(Skills, Subagents, Rules, Hooks, Memory, CLAUDE.md)은 소프트웨어 아키텍처 패턴과 구조적으로 유사하다. 이 대응 관계를 이해하면 Claude Code 설정을 체계적으로 설계하고 확장할 수 있다.

## 상세 설명

### 아키텍처 매핑

| Claude Code | 아키텍처 패턴 | 설명 |
|---|---|---|
| CLAUDE.md (루트) | Composition Root | 모든 설정을 연결하는 엔트리포인트 |
| .claude/rules/ (경로 스코프) | Route-based Middleware | paths frontmatter로 특정 파일 작업 시에만 활성화 |
| Skills (SKILL.md + references/) | Microservice / Feature Module | 단일 책임, 자체 의존성, 정의된 인터페이스. on-demand 로딩 |
| Subagents | Bounded Context (DDD) | 격리된 실행 컨텍스트 + 고유 시스템 프롬프트 + 최소 권한 도구 |
| Auto Memory (MEMORY.md + topic files) | Event Store | 세션 간 영속 상태. 인덱스(MEMORY.md) + 상세(topic files) |
| Hooks (PreToolUse 등) | AOP / Middleware Pipeline | 횡단 관심사(검증, 로깅)를 라이프사이클 포인트에 주입 |
| @import | Dependency Injection | 외부 파일을 명시적으로 선언하여 세션 시작 시 해결 |

### 핵심 패턴 유사성

**Progressive Disclosure = Lazy Loading**
Skill은 3단계 로딩: 메타데이터(~100 토큰) → SKILL.md(<5K 토큰) → reference files(on-demand). FSD의 feature slice lazy loading과 동일한 구조.

**Skill 메타 도구 = API Gateway**
단일 "Skill" 도구가 LLM 추론으로 개별 skill에 라우팅. 마이크로서비스의 API Gateway 패턴.

**SKILL.md의 관심사 분리**
"프로세스는 SKILL.md에, 컨텍스트는 reference 파일에" — Clean Architecture의 Use Case(application layer)와 Entity(domain layer) 분리와 동일.

**Subagent = Bounded Context**
고유 시스템 프롬프트(유비쿼터스 언어) + 도구 제한(인터페이스 계약) + 권한 모드(보안 경계) + 선택적 메모리(자체 데이터 스토어).

### 파일 크기 제한 및 분리 정책

| 파일 | 권장 크기 | 초과 시 영향 |
|---|---|---|
| CLAUDE.md | 200줄 이하 | 400줄 넘으면 규칙 준수율 92% → 71% 하락 |
| CLAUDE.md (경고) | 40KB | Claude Code가 성능 경고 표시 |
| MEMORY.md | 200줄 / 25KB | 초과분 세션 시작 시 미로드 |
| SKILL.md | 500줄 이하 | 상세 내용은 reference 파일로 분리 |
| Skill description | 250자 | 초과분 truncate |

### 분리 전략 (우선순위 순)

**1. `.claude/rules/` — 경로 스코프 규칙 (컨텍스트 절약 최대)**
`paths` frontmatter로 해당 파일 작업 시에만 로드. 실제 토큰 소비를 줄이는 가장 효과적인 방법.

**2. Skills로 승격 — on-demand 로딩**
가끔 필요한 도메인 지식/워크플로우를 `.claude/skills/`로 이동. 호출 시에만 로드.

**3. `@import` 디렉티브 — 유지보수성 향상**
세션 시작 시 전부 확장됨. 총 토큰은 줄지 않지만 파일 관리가 편해짐.

**4. Memory topic 파일 — 인덱스/상세 분리**
MEMORY.md는 인덱스만, 상세는 별도 topic 파일로. 필요 시에만 로드.

> **핵심**: rules의 `paths` 스코프와 skills의 on-demand 로딩만이 실제 컨텍스트를 절약한다. @import 분리는 유지보수성만 개선.

### 커뮤니티 논의

- **GitHub anthropics/claude-code#2766**: CLAUDE.md가 44.7K자로 성장한 사례. `.claude/docs/`로 분리 + @import으로 해결
- **HN 토론**: "60~120줄이 적정. 복잡한 에이전트는 800줄+도 있지만, 인덱스 문서 + 짧은 파일들로 리팩토링하는 것이 트렌드"
- **커뮤니티 공통 인사이트**: "prompt engineering → context engineering" 전환이 "모놀리스 → 마이크로서비스" 전환과 동일한 구조적 변화

## 실무 적용

현재 프로젝트에서 CLAUDE.md가 커질 경우:
1. 기술 스택별 규칙을 `.claude/rules/`로 분리하고 `paths`로 스코프 지정
2. 반복적 워크플로우(workflow-design, workflow-impl 등)는 이미 skill로 분리되어 있음
3. Memory는 인덱스/토픽 구조로 운영 중

## 출처 / 참고

- [Claude Agent Skills Deep Dive](https://leehanchung.github.io/blogs/2025/10/26/claude-skills-deep-dive/)
- [Skills Architecture Design (ClaudeCN)](https://claudecn.com/en/docs/agent-skills/architecture/)
- [Claude Code Skills Architecture (MindStudio)](https://www.mindstudio.ai/blog/claude-code-skills-architecture-skill-md-reference-files)
- [GitHub anthropics/claude-code#2766](https://github.com/anthropics/claude-code/issues/2766)
- [HN: What's the right size claude.md file?](https://news.ycombinator.com/item?id=45689618)
- [[hooks]] — Claude Code Hooks
- [[mcp-tool-token-consumption]] — MCP 도구와 토큰 소모

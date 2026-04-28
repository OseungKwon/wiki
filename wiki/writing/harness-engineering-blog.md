---
title: "하네스 엔지니어링으로 AI 에이전트의 레거시 복제를 막는 법"
aliases:
  - 하네스 엔지니어링 블로그
  - harness engineering blog
tags:
  - writing
  - blog
  - harness-engineering
  - claude-code
  - hook
  - eslint
  - ai-agent
created: 2026-04-28
updated: 2026-04-28
reviewed: false
---

## 프롬프트만으로는 안 됐다

AI 코딩 에이전트를 팀에 도입하면 처음에는 빠르다. 지시를 주면 코드가 나오고, 파일이 생기고, 커밋이 만들어진다. 그런데 며칠 지나면 문제가 보인다. 에이전트가 recoil을 import하고 있다. 팀에서 이미 걷어내기로 한 라이브러리인데. 기존 코드에 남아있으니까 그걸 보고 그대로 따라 쓴 것이다.

금지한 컴포넌트를 쓰고, 아키텍처 경계를 넘는 import를 만들고, inline style을 남긴다. 프롬프트에 "recoil 쓰지 마"라고 적어도 소용없다. 에이전트는 레거시 코드를 먼저 읽고, 거기서 패턴을 복제한다. 프롬프트보다 코드베이스의 중력이 더 세다.

하네스 엔지니어링은 이 문제를 프롬프트가 아니라 환경으로 해결한다. 에이전트의 실행 환경 자체에 제약과 안내를 심어서, 잘못된 코드가 만들어지기 어렵게 하는 방식이다.

## 하네스 엔지니어링

Martin Fowler 블로그의 [Harness engineering for coding agent users](https://martinfowler.com/articles/harness-engineering.html) 글에서 이렇게 정의한다. **Agent = Model + Harness**. 모델을 제외한 나머지 전부, 그러니까 린터, 타입체커, 테스트, 훅 스크립트, 문서, CI 파이프라인까지 에이전트를 감싸는 환경 전체가 하네스다.

프롬프트 엔지니어링이 "무엇을 해라"라는 지시라면, 컨텍스트 엔지니어링은 "이걸 보고 해라"라는 참고 자료 제공이고, 하네스 엔지니어링은 애초에 잘못된 길로 못 가게 환경을 설계하는 것이다. 포함 관계로 보면 하네스가 가장 넓고, 그 안에 컨텍스트가, 그 안에 프롬프트가 들어간다.

하네스는 두 축으로 구성된다.

하나는 **안내**다. 에이전트가 작업을 시작하기 전에 올바른 방향을 제시하는 것. 아키텍처 문서, 코딩 컨벤션, 템플릿, CLAUDE.md 같은 지침 파일이 여기에 해당한다. Fowler 글에서는 이걸 Guides, 혹은 피드포워드 제어라고 부른다.

다른 하나는 **검증**이다. 에이전트가 작업한 결과를 확인하는 것. Fowler 글이 여기서 중요한 구분을 하나 하는데, 결정론적 검증(Computational)과 추론적 검증(Inferential)이다. 결정론적 검증은 린터, 타입체커, 테스트처럼 같은 입력이면 항상 같은 결과가 나온다. 추론적 검증은 AI 코드 리뷰처럼 비결정론적이고 의미론적이다. 둘 다 필요하지만, 빠르고 명확한 피드백을 주는 결정론적 검증을 먼저 최대한 활용하는 게 좋다.

차단만이 하네스의 전부는 아니다. 올바른 길을 안내하는 것과, 벗어났을 때 잡아주는 것이 같이 있어야 한다.

## 프로젝트 상황

모노레포 구조의 React 백오피스 프로젝트다. `packages/service-*` 단위로 서비스가 나뉘고, 공통 모듈은 `packages/common-*`에 있다. 레거시도 상당하다. recoil 기반 상태 관리, 구버전 아토믹 컴포넌트, 직접 axios 호출 같은 코드가 아직 남아있다.

새 서비스를 만들 때는 이런 레거시 패턴을 쓰면 안 된다. React Query, 새 UI 컴포넌트 라이브러리, ApiService 래퍼를 써야 한다. 사람이 코드를 쓸 때는 PR 리뷰에서 잡으면 된다. 그런데 AI 에이전트가 코드를 쓸 때는? 에이전트는 기존 코드를 참고해서 패턴을 복제하니까, 레거시가 남아있는 한 계속 잘못된 코드를 만든다.

## Claude Code 훅에 하네스 붙이기

하네스를 Claude Code의 훅 시스템에 통합했다. `.claude/settings.json`에 훅을 등록하면, 에이전트가 파일을 수정할 때마다 검증 스크립트가 돌아간다.

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{
        "type": "command",
        "command": ".claude/hooks/pre-edit-write.sh",
        "timeout": 5
      }]
    }],
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{
        "type": "command",
        "command": ".claude/hooks/post-edit-write.sh",
        "timeout": 30
      }]
    }]
  }
}
```

우리 하네스는 안내와 결정론적 검증 위주로 구성되어 있다. 추론적 검증(AI 리뷰 등)은 아직 넣지 않았다. 결정론적 도구만으로도 잡을 수 있는 규칙 위반이 많았기 때문에, 먼저 그쪽을 탄탄하게 깔고 시작한 것이다.

하네스는 설계, 구현, UI 세 단계에 걸쳐 동작한다.

### 설계 단계: 산출물 구조 검증

설계 단계에서는 설계 문서의 구조를 검증한다. `validate-output.sh`가 도메인 정의서와 기능 목록이 존재하는지, 필수 섹션이 빠지지 않았는지, `[NEEDS CLARIFICATION]` 태그가 남아있지 않은지 확인한다.

이 검증이 왜 필요한가 하면, 에이전트한테 설계를 시키면 중간 단계를 통째로 건너뛰는 경우가 있기 때문이다. 실제로 그랬다. 도메인 정의도 안 끝났는데 Task 문서를 전부 생성해버린 적이 있다. 구조 검증이 이런 폭주를 막는다.

### 구현 단계: 방어선이 가장 두꺼운 곳

구현 단계에 하네스가 가장 많이 들어간다.

**먼저 안내부터.** 에이전트가 구현을 시작할 때 순서를 강제한다. Task 문서를 읽고, 템플릿을 확인한 뒤에 코드를 작성해야 한다. 이 순서를 밟기 전에 기존 서비스 코드를 탐색하면 안 된다. 레거시 코드를 먼저 보면 그 패턴을 그대로 복제한다. 템플릿을 먼저 보여줘서, 에이전트가 참고할 코드의 기준점을 올바른 쪽으로 잡아주는 것이다.

**그다음 검증.** 검증은 세 단계로 쌓여있다.

첫 번째는 사전 차단이다. 에이전트가 파일을 수정하기 전에 pre-edit-write.sh가 내용을 검사한다. 서비스 패키지 하위의 `.ts/.tsx` 파일에서 금지 패턴을 감지하면 도구 호출 자체를 차단한다.

```bash
REGEXES=(
  "from ['\"]recoil"
  "from ['\"]@common/atomics"
  "from ['\"]@common/icons"
  "axios[.](get|post|put|delete|patch)"
)
```

이 패턴이 새로 추가되는 코드에서 감지되면 `deny`를 반환한다. 에이전트의 Edit/Write 호출이 아예 실행되지 않는다. 파일에 금지 패턴이 기록되는 것 자체가 불가능하다.

두 번째는 사후 교정이다. 파일이 수정된 직후에 post-edit-write.sh가 ESLint `--fix`를 돌린다. 아키텍처 경계 위반, 레거시 import, 금지 패턴, SCSS 디자인 토큰을 자동 수정한다. 파일 네이밍이나 코드 배치 같은 구조적 문제는 자동 수정이 안 되니까 피드백만 준다.

사전 차단은 문 앞에서 막고, 사후 교정은 들어온 뒤에 바로 고친다. 이 두 레이어가 겹치니까 대부분의 위반은 사람 손을 안 탄다. 전부 결정론적 검증이다. 밀리초에서 초 단위로 돌아간다.

세 번째는 Task 게이트다. Task 하나가 끝나면 `task-complete.sh`가 `--fix` 없이 순수 검증만 돌린다. tsc, ESLint, 파일 네이밍, SCSS 토큰, 코드 배치를 전부 확인하고, 하나라도 실패하면 다음 Task로 못 넘어간다. 여기에 "1 Task = 1 사용자 확인" 규칙도 걸어뒀다. Task가 끝날 때마다 에이전트가 반드시 멈추고 사람의 확인을 기다린다. 이게 없으면 에이전트가 Task를 연속으로 처리하면서 방향이 틀어져도 멈추지 않는다.

### UI 단계: Figma에서 코드로

Figma 디자인을 코드로 변환하는 단계에도 하네스가 있다.

Figma MCP의 `get_design_context` 응답은 양이 많다. 테스트해보니 약 7만 토큰이 나왔다. 이걸 그대로 에이전트에 넘기면 토큰 낭비고 변환 정확도도 떨어진다.

post-figma-design.sh 훅이 응답 직후 실행되어 Figma API로 인스턴스를 식별하고, 공통 컴포넌트로 승격시키면서 무손실 압축을 한다. 7만 토큰이 2만 토큰으로 줄었다. 토큰 절감도 의미가 있지만, 더 좋은 건 공통 컴포넌트를 명시적으로 매핑하니까 에이전트가 디자인 시스템의 컴포넌트를 제대로 쓰게 된다는 것이다.

Figma 인스턴스와 프로젝트 컴포넌트의 매핑 표를 만든 뒤, 코드 생성이 끝나면 `grep`으로 매핑 표의 모든 컴포넌트가 실제 코드에 반영됐는지 검증한다. 매핑해놓고 구현에서 `<div>`나 인라인 스타일로 바꿔치기하는 패턴이 있었기 때문이다.

## 하네스가 잡는 것들

ESLint와 Stylelint가 자동으로 고칠 수 있는 것들이 있고, 에이전트가 피드백을 보고 직접 고쳐야 하는 것들이 있다. 구분이 중요하다.

자동 수정이 되는 규칙은 이렇다. 서비스 패키지 사이의 import를 차단하는 아키텍처 경계 규칙, recoil이나 구버전 컴포넌트 같은 레거시 import 차단, 직접 axios 호출이나 inline style 같은 금지 패턴 차단, 그리고 색상이나 폰트를 변수 대신 하드코딩하는 SCSS 규칙. 이런 건 ESLint/Stylelint `--fix`가 저장 직후 알아서 고친다.

자동 수정이 안 되는 규칙도 있다. 파일 이름이 `*.page.tsx`, `*.view.tsx` 같은 컨벤션을 따르는지, useQuery 같은 훅이 `custom-hooks/` 디렉토리 안에만 있는지, `types/`나 `views/` 같은 필수 디렉토리가 갖춰졌는지. 이런 건 파일 구조를 이해해야 하는 문제라 기계적으로 고칠 수가 없다. 피드백으로 알려주고 에이전트가 직접 수정한다.

## 실행 흐름

```
에이전트가 Edit/Write 호출
  ↓
[사전] pre-edit-write.sh (5초)
  → 금지 패턴 감지 시 deny, 도구 호출 자체 취소
  ↓
[사후] post-edit-write.sh (30초)
  → ESLint --fix 자동 수정 + 네이밍/배치 피드백
  ↓
Task 구현 완료
  ↓
[게이트] task-complete.sh
  → tsc + ESLint(fix 없음) + 전체 규칙 검증
  → 통과해야 다음 Task
  ↓
Feature 전체 완료
  ↓
[통합] phase-5-final.sh
  → tsc + ESLint + Stylelint + build 전부 0건
  ↓
[런타임] chrome-devtools MCP
  → 브라우저에서 콘솔 에러 0건 확인
```

앞쪽 관문일수록 자동화 수준이 높다. 사전 차단과 사후 교정은 사람이 아무것도 안 해도 돌아간다. Task 게이트부터 사람이 확인한다. Fowler 글의 표현을 빌리면 "품질을 좌이동(shift left)" 시킨 것이다. 문제를 가능한 한 이른 시점에 잡는다.

## 적용하면서 알게 된 것들

처음에는 사후 교정만 있었다. ESLint `--fix`가 대부분 잡으니까 그걸로 될 줄 알았다. 그런데 금지 패턴이 파일에 한번 기록되고 나면, 에이전트가 다음 턴에서 그 파일을 읽을 때 교정 전 패턴을 이미 본 상태가 될 수 있다. 사전 차단을 따로 만든 건 이 때문이다. 파일에 기록되는 것 자체를 막으면 오염 가능성이 없어진다.

실패 패턴 기록도 효과가 있었다. `failure-patterns.md`에 이전 실패를 정형화된 형태로 적어두면, 에이전트가 구현을 시작하기 전에 이걸 읽는다. Figma 매핑 누락이나 Provider 주입 누락 같은 실수가 반복되지 않았다. 이건 안내 쪽 도구다. 에이전트에게 "이전에 이런 실수가 있었다"는 맥락을 미리 주는 것이다.

컨텍스트 유실 문제도 있다. AI 에이전트와의 대화에는 컨텍스트 윈도우 한계가 있어서, 긴 작업 중에 컨텍스트가 날아가면 처음부터 다시 해야 한다. 진행 상태 파일에 Phase/Task 상태를 기록해두면 컨텍스트가 초기화되어도 이어서 작업할 수 있다. Anthropic의 [Harness design for long-running application development](https://www.anthropic.com/engineering/harness-design-long-running-apps) 글에서 이걸 "컨텍스트 불안감(Context Anxiety)"이라고 부른다. 컨텍스트 한계에 가까워지면 에이전트가 작업을 조기 종료하려는 경향이 생기는데, 구조화된 상태 파일로 세션 간 인수인계를 하면 이 문제를 완화할 수 있다.

그리고 모든 걸 자동 수정하려고 하면 안 된다. import 경로나 디자인 토큰은 기계적으로 고칠 수 있지만, 파일 네이밍이나 코드 배치는 디렉토리 구조를 이해해야 한다. 이런 건 피드백만 주고 에이전트가 직접 고치게 했다. 억지로 자동화하면 오히려 잘못 고친다.

## 아직 안 한 것들

Fowler 글에서는 하네스를 세 범주로 나눈다. 유지보수성 하네스(린터, 테스트), 아키텍처 피트니스 하네스(성능, 관찰성), 행동 하네스(기능적 정확성). 우리가 만든 건 대부분 유지보수성 하네스에 해당한다. 아키텍처 피트니스나 행동 하네스는 아직 손대지 못했다. Fowler 글에서도 행동 하네스에 대해 "AI가 생성한 테스트에 대한 신뢰가 아직 충분하지 않다"고 적어뒀는데, 솔직히 동의한다.

추론적 검증(AI가 AI의 코드를 리뷰하는 방식)도 고려는 했지만 아직 넣지 않았다. 결정론적 검증으로 잡을 수 있는 범위가 생각보다 넓었고, 추론적 검증은 비결정론적이라 오탐을 어떻게 처리할지 아직 정리가 안 됐다.

## 프롬프트를 안 쓴다는 게 아니다

좋은 프롬프트가 있어야 에이전트가 방향을 잡는다. 하네스는 그 방향에서 이탈하지 못하게 환경에 울타리를 치는 것이다. 프롬프트만으로 에이전트를 제어하겠다는 건, 내비게이션만 켜놓고 가드레일 없는 도로에 차를 올리는 것과 비슷하다.

모델 성능이 올라가면 하네스가 필요 없어질까? Anthropic 글에서 이에 대한 흥미로운 관점을 제시한다. "모든 하네스 구성 요소는 모델이 혼자서는 못 하는 것에 대한 가정을 인코딩한 것"이고, 모델이 좋아지면 그 가정을 정기적으로 재검증해야 한다고. Opus 4.5에서는 스프린트 단위 분해가 필요했는데 Opus 4.6에서는 필요 없어진 사례를 들면서, 하네스도 모델과 함께 진화해야 한다고 말한다. 하네스가 영구적 인프라가 아니라 모델의 현재 한계에 맞춰 조정되는 스캐폴딩이라는 건 기억해둘 만하다.

하네스의 구현은 대부분 쉘 스크립트와 ESLint 설정이다. 이미 린터와 타입체커를 쓰고 있다면, 그걸 에이전트의 훅 시스템에 연결하는 것만으로 시작할 수 있다.

## 출처 / 참고

- [Harness engineering for coding agent users](https://martinfowler.com/articles/harness-engineering.html) - Martin Fowler (Birgitta Böckeler). "Agent = Model + Harness" 정의와 Guides/Sensors 프레임워크
- [Harness design for long-running application development](https://www.anthropic.com/engineering/harness-design-long-running-apps) - Anthropic. 다중 에이전트 하네스 설계와 컨텍스트 불안감(Context Anxiety) 개념
- [하네스 엔지니어링: 에이전트 우선 세계에서 Codex 활용하기](https://openai.com/ko-KR/index/harness-engineering/) - OpenAI
- [Everything I Learned About Harness Engineering and AI Factories in San Francisco](https://escape.tech/blog/everything-i-learned-about-harness-engineering-and-ai-factories-in-san-francisco-april-2026/) - escape.tech. 7계층 최소 아키텍처
- [Software 3.0 시대, Harness를 통한 조직 생산성 저점 높이기](https://toss.tech/article/harness-for-team-productivity) - Toss
- [프롬프트와 컨텍스트를 넘어, AI 에이전트를 위한 하네스 엔지니어링](https://madplay.github.io/post/harness-engineering)
- [everything is a ralph loop](https://ghuntley.com/loop/)
- 관련: [[hooks|Claude Code Hooks]]

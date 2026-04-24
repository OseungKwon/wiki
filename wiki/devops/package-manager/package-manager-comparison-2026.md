---
title: JavaScript 패키지 매니저 비교 (2026)
aliases:
  - npm vs yarn vs pnpm
  - 패키지 매니저 비교 2026
tags:
  - package-manager
  - npm
  - yarn
  - pnpm
  - bun
created: 2026-04-24
updated: 2026-04-24
reviewed: false
---

## 정의
npm, yarn, pnpm, bun 네 가지 JavaScript 패키지 매니저의 2026년 4월 기준 최신 버전별 특징과 장단점 비교.

## 최신 버전

| 패키지 매니저 | 최신 안정 버전 | 차기 버전 |
|---|---|---|
| npm | v11.13.0 | - |
| yarn | v4.9.4 (Berry) | - |
| pnpm | v10.33.0 | v11 RC (2026.04) |
| bun | v1.3.x | v2.0 (2026 하반기 예정) |

## 상세 설명

### npm v11

Node.js에 기본 번들. 별도 설치 불필요.

**v11 주요 변경점:**
- 대규모 설치 시 v10 대비 **65% 빠름** (847개 패키지 기준 90초 → 35초 이하)
- `npm trust` — 여러 패키지에 trusted publishing 일괄 설정
- `--allow-git` 플래그로 git 의존성 명시적 제어 (v12에서 `--allow-git=none` 기본값 예정)
- `--ignore-scripts`가 `prepare` 포함 모든 lifecycle script에 적용
- `npm init` 시 ESM/CJS `type` 선택 프롬프트 추가

**장점:** 제로 설정, 가장 넓은 생태계, 호환성 문제 거의 없음
**단점:** 설치 속도가 pnpm/bun 대비 느림, 프로젝트마다 node_modules 전체 복제, phantom dependency 문제 잔존

### yarn v4.9 (Berry)

**Zero-Install 사실상 폐기:**
- `yarn init`으로 새 프로젝트 생성 시 Zero-Install이 **기본 비활성화**
- `.yarn/cache`가 기본 `.gitignore`에 포함 → 캐시가 git에 커밋되지 않음
- Zero-Install 자체는 deprecated가 아니지만, 수동 `.gitignore` 수정 필요
- repo 크기 폭증 문제가 지적되어 사실상 권장하지 않는 방향

**PnP 현실:**
- PnP가 여전히 기본 `nodeLinker`이지만, 호환성 문제로 많은 팀이 `nodeLinker: node-modules`로 변경
- TypeScript, ESLint, Jest 등에서 PnP 관련 의존성 해석 이슈 지속 (v4.5.1까지 버그 존재)
- `node_modules`를 전제하는 도구가 많아 PnP 도입 시 `packageExtensions`, SDK 설정 등 추가 작업 필요
- `nodeLinker: node-modules`로 쓰면 yarn을 써야 할 이유가 크게 약해짐

**v4 고유 강점:**
- JavaScript 기반 constraints 엔진 — 모노레포 정책 강제 가능 (고유 기능)
- Hardened Mode — lockfile 무결성 검증으로 supply chain 공격 방어
- 플러그인 아키텍처로 확장 가능
- Corepack 통합 (`packageManager` 필드로 버전 고정)

**단점:** PnP 호환성이 최대 장벽, 설정 파일 다수, 신규 채택률이 pnpm에 밀리는 추세

### pnpm v10.33 (+ v11 RC)

2023~2025 사이 Vue, Nuxt, Vite 등 주요 프로젝트가 채택. 2026년 신규 프로젝트의 사실상 표준.

**v10 핵심:**
- Content-Addressable Storage + hard link → 디스크 60~70% 절약
- `--filter` 최적화 (수백 패키지 모노레포에서 100ms 이내 해석)
- `allowBuilds` — 세분화된 build script 실행 권한
- CI 기본 스크립트 비활성화 (공급망 보안)
- Global Virtual Store (`enableGlobalVirtualStore: true`)

**v11 RC (2026.04 발표):**
- **SQLite 기반 store index** — 개별 JSON → 단일 SQLite DB로 filesystem syscall 대폭 감소
- **ESM 전용 배포** — pnpm 자체가 pure ESM 전환
- **Node.js 22+ 필수** (18~21 지원 중단)
- `minimumReleaseAge` 기본값 1일 — 신규 publish 후 24시간 대기 (typosquatting 방어)
- `.npmrc`에서 auth/registry만 읽고, 나머지는 `pnpm-workspace.yaml`로 이동
- 글로벌 설치 격리 — `pnpm add -g`마다 독립 디렉토리/lockfile

**장점:** npm 대비 3~5배 빠른 cold install, 디스크 효율 최고, phantom dependency 원천 차단, Turborepo 조합이 업계 표준, npm과 CLI 유사하여 전환 비용 낮음
**단점:** symlink/hard link 기반 일부 도구 호환 이슈, v11부터 Node 22+ 필수

### bun v1.3

Zig로 작성된 올인원 JS 런타임. 패키지 매니저 내장.

**설치 속도:** npm 대비 최대 25배, pnpm 대비 4~5배 빠름 (847개 패키지: npm 32초 vs bun 1.2초)
**한계:** Node.js 완전 호환 미완, 프로덕션 안정성 검증 중, 생태계 가장 작음

## 종합 비교

| 기준 | npm v11 | yarn v4.9 | pnpm v10 | bun v1.3 |
|---|---|---|---|---|
| Cold install 속도 | ★★★ | ★★★ | ★★★★ | ★★★★★ |
| 디스크 효율 | ★★ | ★★★ | ★★★★★ | ★★★★ |
| 모노레포 | ★★ | ★★★ | ★★★★★ | ★★★ |
| 보안 기본값 | ★★★ | ★★★★ | ★★★★★ | ★★★ |
| 호환성 | ★★★★★ | ★★★ | ★★★★ | ★★★ |
| 학습 곡선 | 쉬움 | 높음 | 낮음 | 낮음 |
| 채택 추세 (2026) | 유지 | 하락 | 상승 | 성장 중 |

## 추천 시나리오

| 상황 | 추천 |
|---|---|
| 설정 최소화, 범용 | npm |
| 모노레포, CI 최적화, 디스크 절약 | pnpm |
| 기존 yarn 프로젝트 유지보수 | yarn (`nodeLinker: node-modules`) |
| 극한의 설치 속도, 실험적 환경 | bun |
| 신규 프로젝트 (2026 기준) | **pnpm** |

## 출처 / 참고
- [pnpm 11 RC - InfoQ](https://www.infoq.com/news/2026/04/pnpm-11-rc-release/)
- [npm v11 Changelog](https://docs.npmjs.com/cli/v11/using-npm/changelog/)
- [Yarn 4.0 Release](https://yarnpkg.com/blog/release/4.0)
- [pnpm vs Bun 2026](https://www.pkgpulse.com/blog/pnpm-vs-bun-2026)
- [[pnpm-store-and-comparison|pnpm-store와 패키지 매니저 비교]] — pnpm store 구조 심화

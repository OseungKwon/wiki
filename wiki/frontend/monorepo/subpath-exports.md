---
title: Package Subpath Exports로 의존성 지옥 해결하기
aliases:
  - subpath exports
  - package exports
  - 서브패스 익스포트
tags:
  - node-js
  - package-json
  - monorepo
  - module-system
created: 2026-04-03
updated: 2026-04-03
reviewed: false
---

## 정의

`package.json`의 `exports` 필드를 사용해 패키지의 진입점(entrypoint)을 여러 서브패스로 분리하는 기법. 단일 `index.ts`에서 모든 것을 export할 때 발생하는 불필요한 의존성 로딩, 타입 충돌, 서버/클라이언트 로직 혼재 문제를 해결한다.

## 상세 설명

### 문제: 단일 진입점의 의존성 지옥

```ts
// 모든 것을 index.ts에서 export
import { useCustomHook, buildImageUrl, ServerUtil } from '@my-org/utils'
// → rrv7, next, vite 등 모든 의존성이 한꺼번에 resolve됨
// → 타입 충돌, 서버/클라이언트 로직 혼재
```

패키지 내부에서 Next.js, React Router v7, Vite 등 프레임워크별 로직이 있는데, 단일 `index.ts`에서 모두 re-export하면:
- 소비자가 사용하지 않는 프레임워크의 의존성까지 resolve됨
- 서버 전용 코드가 클라이언트 번들에 포함되거나 그 반대
- 프레임워크 간 타입 충돌 발생

### 해결: `exports` 필드로 서브패스 분리

```jsonc
// packages/shared-utils/package.json
{
  "name": "@my-org/utils",
  "exports": {
    ".": "./src/index.ts",            // 순수 유틸만
    "./react": "./src/react/index.ts", // React 공통
    "./rrv7": "./src/rrv7/index.ts",   // React Router v7 전용
    "./next": "./src/next/index.ts",   // Next.js 전용
    "./server": "./src/server/index.ts" // 서버 전용
  }
}
```

사용 측:
```ts
import { buildImageUrl } from '@my-org/utils'          // 공통만
import { useCustomHook } from '@my-org/utils/rrv7'     // rrv7 의존성만
import { getServerSession } from '@my-org/utils/next'   // next 의존성만
```

### 디렉토리 구조

```
packages/shared-utils/src/
├── index.ts          # 순수 유틸 (의존성 없음)
├── react/index.ts    # React 공통 (react만 peer dep)
├── rrv7/index.ts     # React Router v7 전용
├── next/index.ts     # Next.js 전용
└── server/index.ts   # 서버 전용 (Node API 등)
```

### peerDependencies + optional 선언

```jsonc
{
  "peerDependencies": {
    "react": "^18 || ^19",
    "react-router": "^7.0.0",
    "next": ">=14"
  },
  "peerDependenciesMeta": {
    "react-router": { "optional": true },
    "next": { "optional": true }
  }
}
```

소비자가 해당 서브패스를 사용할 때만 peer dep 설치가 필요하도록 `optional: true`로 선언한다.

### TypeScript 지원

모노레포 내부 패키지라면 번들러 설정으로 해결되는 경우도 있지만, `typesVersions`를 명시하면 어떤 환경에서든 안전하다:

```jsonc
{
  "typesVersions": {
    "*": {
      "rrv7": ["./src/rrv7/index.ts"],
      "next": ["./src/next/index.ts"],
      "server": ["./src/server/index.ts"]
    }
  }
}
```

### 조건부 exports (환경별 분기)

Node.js 12.11+에서 지원하는 conditional exports로 환경별 자동 분기도 가능하다:

```jsonc
{
  "exports": {
    ".": {
      "import": "./dist/esm/index.js",
      "require": "./dist/cjs/index.js",
      "types": "./dist/types/index.d.ts"
    }
  }
}
```

## 실무 적용

### 마이그레이션 전략

1. 기존 `index.ts`는 유지하되 순수 유틸만 남기고 프레임워크 의존 코드를 제거
2. 프레임워크별 서브패스를 추가하고, 제거한 코드를 각 서브패스로 이동
3. 소비자 코드의 import 경로를 `@my-org/utils` → `@my-org/utils/rrv7` 등으로 변경
4. ESLint `no-restricted-imports`로 잘못된 import 방지

```js
// eslint 설정 예시 (React Router 기반 앱에서)
'no-restricted-imports': ['error', {
  paths: [{
    name: '@my-org/utils/next',
    message: '이 앱은 React Router 기반입니다. @my-org/utils/rrv7를 사용하세요.'
  }]
}]
```

## 출처 / 참고

- [Node.js Packages - Subpath exports](https://nodejs.org/api/packages.html#subpath-exports)
- [TypeScript - Package.json exports](https://www.typescriptlang.org/docs/handbook/modules/reference.html#packagejson-exports)
- [[pnpm-store-and-comparison|pnpm Store와 패키지 매니저 비교]]

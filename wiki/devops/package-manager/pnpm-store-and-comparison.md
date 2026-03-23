---
title: pnpm-store와 패키지 매니저 비교
aliases:
  - pnpm store
  - npm vs pnpm vs yarn
tags:
  - pnpm
  - npm
  - yarn
  - package-manager
  - node
created: 2026-03-19
updated: 2026-03-19
---

## 정의
pnpm-store는 pnpm 패키지 매니저의 콘텐츠 주소 지정 저장소(content-addressable store)로, 다운로드한 모든 패키지를 중앙에 한 벌만 보관하고 하드링크로 각 프로젝트에 연결하는 구조다.

## 상세 설명

### pnpm-store 동작 원리
- 패키지를 최초 설치 시 글로벌 store(`~/.pnpm-store`)에 저장
- 프로젝트의 `node_modules/.pnpm/`에는 store로의 **하드링크**가 생성됨
- 10개 프로젝트가 같은 패키지를 쓰면 디스크에는 1벌만 존재

### node_modules 구조 비교

**npm/yarn (flat 구조)**:
```
node_modules/
  express/
  body-parser/    ← express의 의존성인데 직접 접근 가능 (유령 의존성)
  cookie/
```

**pnpm (심볼링크 + 격리 구조)**:
```
node_modules/
  .pnpm/          ← 실제 패키지 (하드링크)
  express -> .pnpm/express@4.18.2/node_modules/express
```
- `package.json`에 선언한 것만 접근 가능
- 선언 안 한 패키지를 import하면 에러 → **유령 의존성(phantom dependency) 방지**

### 패키지 매니저별 특징

| | npm | yarn classic | yarn berry (PnP) | pnpm |
|---|---|---|---|---|
| **저장 방식** | 프로젝트별 복사 | 프로젝트별 복사 | zip 아카이브 (node_modules 없음) | 글로벌 store + 하드링크 |
| **디스크 효율** | 낮음 | 낮음 | 높음 | 높음 |
| **설치 속도** | 보통 | 빠름 | 매우 빠름 | 빠름 |
| **유령 의존성** | 허용 | 허용 | 차단 | 차단 |
| **호환성** | 높음 | 높음 | 낮음 (PnP 미지원 라이브러리 존재) | 높음 |

### .pnpm-store를 .gitignore에 추가하는 이유
프로젝트 루트에 로컬 store가 생성될 수 있는데, 이는 빌드 아티팩트이므로 git에 포함되면 안 된다.

## 출처 / 참고
- [pnpm 공식 문서 - Store](https://pnpm.io/store)
- [pnpm 공식 문서 - Motivation](https://pnpm.io/motivation)

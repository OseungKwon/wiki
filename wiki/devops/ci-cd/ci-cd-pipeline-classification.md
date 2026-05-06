---
title: "CI/CD 파이프라인 도구 분류"
aliases: ["CI CD 분류", "CI CD 도구"]
tags:
  - ci-cd
  - devops
  - github-actions
  - husky
  - jenkins
  - argocd
created: 2026-05-06
updated: 2026-05-06
reviewed: false
---

## 정의
CI(Continuous Integration)는 코드 변경을 자동으로 빌드·테스트하여 검증하는 프로세스이고, CD(Continuous Delivery/Deployment)는 검증된 코드를 자동으로 배포 환경에 전달·배포하는 프로세스다.

## 상세 설명

### CI vs CD 구분

| 구분 | 목적 | 핵심 질문 |
|------|------|-----------|
| **CI** | 코드가 정상인지 검증 | "이 코드가 기존 코드를 깨뜨리지 않는가?" |
| **CD** | 검증된 코드를 배포 | "이 코드를 어떻게 사용자에게 전달하는가?" |

### 도구별 분류

| 구분 | 도구 | 역할 |
|------|------|------|
| **CI** | Husky | 커밋/푸시 전 로컬에서 lint, 테스트 등 git hook 실행 |
| **CI** | GitHub Actions | PR/푸시 시 원격에서 빌드, 테스트, 검증 자동 실행 |
| **CD** | Jenkins | Docker 이미지 빌드, 배포 파이프라인 실행 |
| **CD** | ArgoCD | K8s 클러스터에 실제 배포 (GitOps 기반) |

### 파이프라인 흐름

```
Husky (로컬) → GitHub Actions (원격 검증) → Jenkins (빌드) → ArgoCD (배포)
   CI                  CI                        CD              CD
```

### 각 도구의 특징

**Husky (로컬 CI)**
- git hook 기반으로 commit/push 시점에 실행
- lint, 타입 체크, 포맷팅 등 빠른 피드백
- 원격 CI 전에 문제를 사전 차단

**GitHub Actions (원격 CI)**
- PR 생성/푸시 이벤트에 트리거
- 빌드, 유닛/통합 테스트, 코드 커버리지 등 실행
- 원격 환경에서의 재현성 보장

**Jenkins (CD - 빌드)**
- Docker 이미지 빌드 및 레지스트리 푸시
- 배포 파이프라인 오케스트레이션
- 다양한 플러그인으로 커스텀 파이프라인 구성

**ArgoCD (CD - 배포)**
- GitOps 방식으로 K8s 클러스터에 배포
- Git 저장소의 선언적 설정과 클러스터 상태를 동기화
- 자동 롤백, 상태 모니터링 제공

## 출처 / 참고
## 출처 / 참고
- [What is CI/CD? - Red Hat](https://www.redhat.com/en/topics/devops/what-is-ci-cd)
- [CI vs CD vs CD - Atlassian](https://www.atlassian.com/continuous-delivery/principles/continuous-integration-vs-delivery-vs-deployment)
- [GitHub Actions Docs](https://docs.github.com/en/actions)
- [Husky - Git hooks made easy](https://typicode.github.io/husky/)
- [Jenkins User Documentation](https://www.jenkins.io/doc/)
- [Argo CD - Declarative GitOps CD for Kubernetes](https://argo-cd.readthedocs.io/en/stable/)

---
title: "브라우저 쿠키 (HTTP Cookie)"
aliases: [cookie, http-cookie, set-cookie, document.cookie]
tags:
  - browser
  - cookie
  - http
  - security
  - web-api
created: 2026-04-24
updated: 2026-04-24
reviewed: false
---

## 정의

HTTP Cookie는 서버가 `Set-Cookie` 헤더로 브라우저에 전달하는 작은 데이터 조각(최대 4KB)으로, 브라우저가 이후 같은 도메인 요청마다 자동으로 `Cookie` 헤더에 포함하여 전송한다. 무상태(stateless)인 HTTP 프로토콜에서 상태를 유지하는 핵심 메커니즘이다.

## 상세 설명

### 쿠키의 생명주기

| 유형 | 만료 시점 | 설정 방법 |
|------|----------|----------|
| Session Cookie | 브라우저 종료 시 | `Expires`/`Max-Age` 생략 |
| Persistent Cookie | 지정된 만료일 | `Expires=날짜` 또는 `Max-Age=초` |

### Set-Cookie 헤더 속성

```http
Set-Cookie: access_token=abc123; Path=/; Domain=.example.com; Max-Age=3600; Secure; HttpOnly; SameSite=Strict
```

| 속성 | 설명 | 기본값 |
|------|------|--------|
| `Domain` | 쿠키가 전송될 도메인. 서브도메인 포함 여부 결정 | 현재 호스트 (서브도메인 제외) |
| `Path` | 쿠키가 전송될 경로 범위 | `/` |
| `Expires` | 만료 일시 (절대 시간, `Date` 형식) | 세션 종료 시 |
| `Max-Age` | 만료까지 남은 초 (`Expires`보다 우선) | — |
| `Secure` | HTTPS 연결에서만 전송 | 미설정 |
| `HttpOnly` | JavaScript(`document.cookie`)에서 접근 불가 | 미설정 |
| `SameSite` | 크로스 사이트 요청 시 전송 정책 | `Lax` (대부분 브라우저) |

### SameSite 정책

| 값 | 동작 | 사용 사례 |
|----|------|----------|
| `Strict` | 크로스 사이트 요청 시 절대 전송하지 않음 | 은행, 결제 등 민감한 인증 |
| `Lax` | top-level 네비게이션(링크 클릭)에서만 전송. form POST, iframe, AJAX에서는 미전송 | 일반 로그인 세션 (기본값) |
| `None` | 항상 전송. 반드시 `Secure`와 함께 사용 | 서드파티 연동, 임베드 |

### JavaScript에서 쿠키 다루기

**네이티브 API** (`document.cookie`):
```typescript
// 읽기 — 모든 쿠키가 하나의 문자열로 반환됨
document.cookie; // "access_token=abc; theme=dark"

// 쓰기 — 한 번에 하나씩 설정
document.cookie = "theme=dark; path=/; max-age=86400; secure";

// 삭제 — Max-Age=0 또는 과거 Expires 설정
document.cookie = "theme=; max-age=0";
```

**라이브러리 사용** (`cookies-next` 등):
```typescript
import { setCookie, getCookie, deleteCookie } from 'cookies-next';

setCookie('access_token', token, { secure: true });
getCookie('access_token');
deleteCookie('access_token');
```

### 보안 고려사항

| 위협 | 방어 속성 |
|------|----------|
| XSS로 토큰 탈취 | `HttpOnly` — JS 접근 차단 |
| 네트워크 스니핑 | `Secure` — HTTPS만 허용 |
| CSRF 공격 | `SameSite=Strict\|Lax` |
| 쿠키 범위 과잉 | `Domain`/`Path` 최소 범위로 설정 |

> 인증 토큰(`access_token`, `refresh_token`)은 반드시 `Secure` + `HttpOnly` + `SameSite=Strict|Lax` 조합으로 설정한다. `HttpOnly`를 사용할 수 없는 클라이언트 환경(SPA)에서는 최소한 `Secure`를 적용한다.

### 쿠키 vs 다른 저장소

| | Cookie | localStorage | sessionStorage |
|---|--------|-------------|----------------|
| 용량 | ~4KB | ~5MB | ~5MB |
| 서버 전송 | 매 요청 자동 전송 | 불가 | 불가 |
| 만료 | 설정 가능 | 영구 | 탭 종료 시 |
| JS 접근 | `HttpOnly`로 차단 가능 | 항상 가능 | 항상 가능 |
| 적합한 용도 | 인증, 세션 | UI 설정, 캐시 | 임시 폼 데이터 |

## 실무 적용

passorder-event 프로젝트에서 게스트 로그인 시 `access_token`과 `refresh_token`을 쿠키에 저장하는 패턴을 사용한다:

```typescript
// guest-login-manager.ts
setCookie(ACCESS_TOKEN_KEY, accessToken, {
  secure: process.env.NEXT_PUBLIC_BUILD_ENV !== 'development',
});
setCookie(REFRESH_TOKEN_KEY, refreshToken, {
  secure: process.env.NEXT_PUBLIC_BUILD_ENV !== 'development',
});
```

개발 환경에서는 `Secure` 비활성화(HTTP localhost 대응), 프로덕션에서는 `Secure` 활성화로 분기한다.

## 출처 / 참고

- [MDN - HTTP Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)
- [MDN - Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)
- [[rendering-pipeline]] — 브라우저 내부 동작 관련

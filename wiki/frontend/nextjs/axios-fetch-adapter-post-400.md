---
title: "Axios Fetch Adapter - Node.js POST 400 에러"
aliases:
  - axios fetch adapter 400
  - fetch adapter post body error
tags:
  - axios
  - fetch
  - next-js
  - node-js
  - troubleshooting
created: 2026-03-26
updated: 2026-03-26
---

## 정의
Next.js 서버 환경에서 axios의 `adapter: 'fetch'` 설정 시, body가 있는 요청(POST/PUT)이 400 Bad Request를 반환하는 문제. Node.js fetch API의 body 전송 방식이 기본 http 모듈과 다른 것이 근본 원인이다.

## 증상
- 서버 컴포넌트(Node.js)에서 POST 요청 시 **400 Bad Request** 반환
- 클라이언트(브라우저)에서 동일 요청 시 정상 동작
- GET 요청은 서버/클라이언트 모두 정상
- 프록시 유무와 관계없이 발생 (로컬 환경에서도 재현됨)

## 원인
axios에 `adapter: 'fetch'`를 설정하면 Node.js의 native fetch API를 사용하게 되는데, 이는 기본 http 모듈과 body 전송 방식이 다르다.

| | 기본 http adapter | fetch adapter (Node.js) |
|---|---|---|
| `Content-Length` | 자동 설정 | **누락** |
| Body 전송 방식 | 일반 전송 | `Transfer-Encoding: chunked` |
| `duplex` 옵션 | 불필요 | `duplex: "half"` 필요 |

Node.js fetch는 body를 스트림으로 전송하면서 `Content-Length`를 설정하지 않고, body 직렬화 방식도 달라 백엔드 서버가 body를 정상 파싱하지 못해 400을 반환한다. 프록시(nginx 등)가 있는 경우 chunked + Content-Length 충돌로 문제가 더 악화될 수 있다.

**브라우저 fetch vs Node.js fetch**: 브라우저의 fetch는 `Content-Length`를 정상 설정하므로 클라이언트에서는 문제가 없다. Node.js의 fetch만 이 동작이 다르다.

### 발견 경위
기존에는 서버 컴포넌트에서 POST 요청을 보내는 케이스가 없었기 때문에 문제가 드러나지 않았다. 최초로 BFF 레이어에서 POST 호출이 발생하면서 발견되었다.

## 해결
**GET 요청에만 fetch adapter를 사용**하고, POST/PUT 등은 기본 http adapter를 사용한다.

```typescript
instance.interceptors.request.use((config) => {
  if (config.method?.toUpperCase() === 'GET') {
    config.adapter = 'fetch'; // Next.js 캐시 활용
  }
  // POST, PUT 등은 기본 http adapter 사용
  return config;
});
```

이 접근이 합리적인 이유:
- Next.js 캐시는 GET에만 의미가 있음 (POST/PUT은 캐시 대상이 아님)
- body가 없는 GET은 `Content-Length` 문제 미발생
- 기본 http adapter는 모든 환경에서 안정적으로 동작

### 대안
- `unstable_cache`로 래핑 — adapter 변경 없이 Next.js 캐시 적용 가능
- GET은 native `fetch`, 나머지는 axios — adapter 문제 자체를 회피
- Content-Length 수동 설정 인터셉터 — 부분적 해결만 가능

## 출처 / 참고
- [axios#7494 - Content-Length missing in fetch adapter POST](https://github.com/axios/axios/issues/7494)
- [axios#6464 - Fetch adapter POST body empty on server-side](https://github.com/axios/axios/issues/6464)
- [axios#6886 - Next.js fetchOptions not passed through](https://github.com/axios/axios/issues/6886)
- [[rsc-async-optimization]]

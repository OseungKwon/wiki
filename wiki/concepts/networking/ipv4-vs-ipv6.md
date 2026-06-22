---
title: IPv4 vs IPv6
aliases: [IPv6, IPv4, EAI_AGAIN, getaddrinfo]
tags:
  - networking
  - ipv4
  - ipv6
  - dns
created: 2026-06-22
updated: 2026-06-22
---

## 정의
IPv4(32비트, 약 43억 주소)와 IPv6(128비트, 약 3.4×10³⁸ 주소)는 인터넷 계층의 주소 체계
표준이다. IPv6는 IPv4의 주소 고갈 문제를 해결하기 위해 등장했으며, 헤더 단순화·자동 설정·
IPSec 내장 등을 함께 도입했다. 두 프로토콜은 직접 호환되지 않아 전환기에는 공존(Dual Stack)한다.

## 상세 설명

### 핵심 비교
| | IPv4 | IPv6 |
|---|---|---|
| 주소 길이 | 32비트 | 128비트 |
| 주소 개수 | 약 43억 (2³²) | 약 3.4×10³⁸ (2¹²⁸) |
| 표기 | 점 10진수 `192.168.0.1` | 콜론 16진수 `2001:db8::1` |
| 헤더 | 가변 길이, 체크섬 있음 | 고정 40바이트, 체크섬 없음 |
| 주소 자동설정 | DHCP 필요 | SLAAC로 자동 가능 |
| 보안 | IPSec 선택 | IPSec 기본 내장 |
| NAT | 주소 부족으로 필수화 | 불필요 (End-to-End 복원) |

### IPv6 표기 축약 규칙
- 각 그룹 앞의 `0` 생략: `0db8` → `db8`
- 연속된 `0` 그룹은 `::`로 한 번만 축약: `2001:db8:0:0:0:0:0:1` → `2001:db8::1`

### 전환 메커니즘 (IPv4 ↔ IPv6 공존)
- **Dual Stack**: 양쪽 동시 운영 (가장 흔함)
- **Tunneling**: IPv6 패킷을 IPv4로 감쌈 (6to4, Teredo)
- **NAT64/DNS64**: IPv6 전용 기기가 IPv4 서버와 통신

### IPv6의 단점 (대부분 "전환기"의 부수효과)
1. **호환성**: IPv4와 직접 통신 불가 → 전환 메커니즘을 추가 운영해야 함 (복잡·비용)
2. **느린 보급**: NAT가 주소 고갈을 버티게 해줘 전환 동기가 약함 (닭-달걀 문제)
3. **가독성**: 16진수 주소는 사람이 외우거나 입력하기 불편, 오타·로그 분석 어려움
4. **운영 비용**: SLAAC·NDP 등 새 체계 학습, 방화벽/ACL 재설정 필요
5. **보안 양날의 검**: NAT가 사라져 모든 기기가 공인 주소를 가짐 → 공격 표면 증가,
   NDP 스푸핑 등 새 벡터, 운영 경험 부족으로 인한 설정 실수

## "IPv6가 느리다"는 말의 진실

프로토콜 자체는 오히려 IPv6가 더 빠르게 설계됐다(고정 헤더·체크섬 제거). "느리다"는
체감은 거의 다 전환기의 부수 효과에서 나왔다.

| "느리다"의 원인 | 프로토콜 탓? | 현재 상태 |
|---|---|---|
| 깨진 IPv6 경로 + 타임아웃 폴백 | ❌ 환경 탓 | Happy Eyeballs로 해결 |
| 터널링(6to4/Teredo) 오버헤드 | ❌ 터널 탓 | 네이티브 IPv6면 무관 |
| 초기 라우팅 미성숙(우회 경로) | ❌ 보급 초기 탓 | 대부분 해소 |
| 40바이트 헤더 | ❌ 오해 | 고정+무체크섬이라 처리 빠름 |

- **Happy Eyeballs (RFC 6555/8305)**: 과거엔 IPv6를 먼저 시도하다 깨지면 타임아웃 후
  IPv4 폴백 → 수 초 지연. 지금은 IPv4·IPv6를 거의 동시 시도해 먼저 응답한 쪽을 사용.
- 실측(APNIC, Meta, Akamai)에선 네이티브 IPv6가 IPv4와 같거나 더 빠른 경우가 많음
  (NAT 생략, 짧은 경로). Meta는 자사망에서 10~15% 빠르다고 보고한 바 있음.

## EAI 에러와 IPv6의 관계

`EAI_*`는 `getaddrinfo()`(이름→주소 변환, DNS 해석)가 내는 에러 코드다(**E**rror **A**ddress **I**nfo).

| 코드 | 의미 |
|---|---|
| `EAI_AGAIN` | DNS 일시적 실패 (재시도 가능) — "lookup timed out" |
| `EAI_NONAME` / `ENOTFOUND` | 해당 이름의 레코드 없음 |
| `EAI_NODATA` | 이름은 있으나 요청 타입(A/AAAA) 레코드 없음 |
| `EAI_FAIL` | DNS 영구 실패 |

**왜 IPv6가 원인으로 지목되나**
- 듀얼스택은 도메인 조회 시 A(IPv4)·AAAA(IPv6)를 둘 다 질의한다.
- 일부 DNS 서버/방화벽이 AAAA 질의에 응답하지 않거나 드롭 → AAAA가 타임아웃 →
  `EAI_AGAIN`. A 레코드는 멀쩡한데 AAAA 때문에 전체 해석이 느려지거나 실패.
- `getaddrinfo`는 family 미지정(`AF_UNSPEC`) 시 양쪽 다 질의. Node.js는 내부적으로
  libuv 스레드풀에서 `getaddrinfo`를 쓴다 → `getaddrinfo EAI_AGAIN <host>`를 자주 봄.

**주의**: `EAI_AGAIN`의 가장 흔한 실제 원인은 그냥 DNS 문제다(서버 무응답, 컨테이너
`/etc/resolv.conf` 오설정, rate limit, 스레드풀 고갈). IPv6는 "AAAA가 막혀 타임아웃"
하는 특정 시나리오에서만 진범이다. → `curl -4` vs `curl -6`로 먼저 확인할 것.

### 진단 / 해결
```bash
# IPv6가 범인인지 확인
dig AAAA example.com        # 타임아웃/실패하는지
dig A    example.com
curl -4 https://example.com # 됨
curl -6 https://example.com # 안 됨 → IPv6 경로/DNS 문제 확정
```

Node.js에서 IPv4 우선/강제 (Node 18+는 기본이 verbatim이라 AAAA가 먼저 올 수 있음):
```js
import dns from 'node:dns';
dns.setDefaultResultOrder('ipv4first'); // A 레코드 우선
```
```bash
node --dns-result-order=ipv4first app.js
```
그 외: OS `gai.conf`로 IPv4 우선, AAAA를 제대로 처리하는 리졸버(8.8.8.8 / 1.1.1.1)
사용, Docker `--dns` 옵션 점검.

## 출처 / 참고
- RFC 8200 (IPv6), RFC 8305 (Happy Eyeballs v2)
- APNIC / Meta / Akamai IPv6 성능 측정 보고
- Node.js `dns` 문서 (`--dns-result-order`)
- 관련 위키: [[network-request-flow]], [[ssh]], [[libuv]]

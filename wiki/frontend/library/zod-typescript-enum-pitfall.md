---
title: Zod와 TypeScript enum / namespace 병합의 함정
aliases: [z.nativeEnum 함정, zod nativeEnum, zod enum namespace]
tags:
  - zod
  - typescript
  - enum
  - validation
  - schema
created: 2026-06-08
updated: 2026-06-08
---

## 정의
`z.nativeEnum`(zod v4에서 deprecated)은 런타임에 enum 객체를 훑어 값 집합을
만든다. TypeScript enum의 reverse mapping과 namespace 병합으로 추가된 함수
멤버가 이 값 집합에 새어들어 검증이 오염될 수 있다. 근본 원인은 zod가 아니라
TS `enum` 키워드의 구조적 특성이며, 권장 대안은 `as const` 객체 + 리터럴 union이다.

## 상세 설명

### namespace 병합 enum이 새는 메커니즘
common-values 레이어는 enum에 라벨/파싱 헬퍼를 같은 이름의 namespace로 병합한다.
TS 컴파일 결과 함수가 enum 객체의 실제 프로퍼티로 붙는다.

```js
AccessLevel = {
  1: "READ", 2: "WRITE",      // 정방향
  READ: 1, WRITE: 2,          // 역방향(reverse mapping)
  toString: ƒ, fromLabel: ƒ,  // namespace 병합 멤버 ← 누수 원인
}
```

`z.nativeEnum`의 reverse-mapping 필터는 `obj[obj[k]]`가 number인지만 본다.
함수 멤버는 `obj[함수] === undefined`라 필터를 통과해 값 집합 `[1, 2, ƒ]`가 된다.

| 키 | obj[k] | obj[obj[k]] | 결과 |
|---|---|---|---|
| "1" | "READ" | 1 (number) | 제외 ✅ |
| "READ" | 1 | "READ" (string) | 유지 → 1 |
| "toString" | ƒ | undefined | **유지 → ƒ** ⚠️ |

순수 enum(예: `DisposableCupStatus`)은 함수 누수가 없어 상대적으로 안전하다.

### 왜 TS enum 키워드 자체가 권장되지 않는가
zod 문서는 TS `enum` 사용을 권장하지 않으며 `as const` + union을 권한다(근거: Matt
Pocock, totaltypescript). 핵심 이유:

1. **숫자 enum은 raw number를 다 받는다** — `logStatus(99)`가 통과. 타입이 제약을
   못 해 런타임 검증으로 떠넘긴다 (zod와 상극인 지점).
2. **reverse mapping으로 런타임 객체가 복잡** — 멤버 3개에 키 6개. nativeEnum이
   필터 로직을 따로 둬야 하는 근본 원인.
3. **nominal typing** — 값이 같아도 다른 enum 타입이면 호환 불가. 패키지 경계·
   monorepo에서 재사용/합성이 막힌다. 리터럴 union은 structural이라 호환됨.
4. **숫자/문자열 enum 동작 불일치** — 같은 문법인데 타입별로 다르게 타입체크.
5. **컴파일 산출물 문제** — enum은 런타임 코드를 생성(erasable 아님), tree-shaking·
   번들 크기에 불리. `const enum`은 isolatedModules(Babel/SWC/esbuild)와 충돌.

### zod 버전별 변화
- **v3**: `z.nativeEnum(MyEnum)`으로 TS enum 처리. `z.enum`은 문자열 리터럴 전용.
- **v4**: `z.nativeEnum` deprecated → `z.enum()`이 TS enum도 직접 수용.
  단 namespace 병합 멤버에 대한 방어는 문서에 명시 없음 → 병합 enum엔 여전히 주의.

## 실무 적용

**권장: `as const` 객체 + 리터럴 union** (라벨/파싱 헬퍼는 enum에 병합하지 말고 분리)
```ts
const AccessLevel = { READ: 1, WRITE: 2 } as const;
type AccessLevel = (typeof AccessLevel)[keyof typeof AccessLevel]; // 1 | 2

const accessLevelSchema = z.union([z.literal(1), z.literal(2)]);
```

**대안 비교**
| 방식 | namespace 누수 | 값 동기화 | 비고 |
|---|---|---|---|
| z.nativeEnum(병합 enum) | ⚠️ 있음 | 자동 | 피할 것 |
| z.nativeEnum(순수 enum) | 없음 | 자동 | OK |
| z.union([literal...]) | 없음 | 수동 | 가장 명시적 |
| z.number().refine | 없음 | 반자동 | 헬퍼 재사용 가능 |
| as const + union (v4) | 없음 | 반자동 | 권장 |

**헬퍼 재사용이 필요할 때: z.number().refine**
```ts
z.number().refine(
  (v): v is AccessLevel => v === AccessLevel.READ || v === AccessLevel.WRITE,
);
```

이 프로젝트는 common-values에서 enum+namespace로 라벨/파싱을 묶는 컨벤션이라
zod 연동 경계에서 마찰이 구조적으로 발생한다. 그 경계에서만 `z.union([z.literal()])`로
명시하거나 헬퍼를 enum에서 떼어내는 절충이 현실적이다.

## 출처 / 참고
- Zod v4 Migration guide — nativeEnum deprecated: https://zod.dev/v4/changelog
- Zod API (Enums, TS enum not recommended 주석): https://zod.dev/api
- zod Discussion #2125 — z.enum breaks with actual enum / Object.values
- Why I don't like TypeScript enums — Matt Pocock: https://www.totaltypescript.com/why-i-dont-like-typescript-enums
- [[Promise.resolve vs new Promise]] — 같은 JS/TS 기초 맥락

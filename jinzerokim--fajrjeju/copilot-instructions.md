## fajrjeju

> Next.js 16에는 breaking changes가 있다 — API, 컨벤션, 파일 구조 모두 훈련 데이터와 다를 수 있다.

# This is NOT the Next.js you know

Next.js 16에는 breaking changes가 있다 — API, 컨벤션, 파일 구조 모두 훈련 데이터와 다를 수 있다.

## 필수 절차

1. 코드 작성 전 `node_modules/next/dist/docs/` 에서 관련 문서 확인
2. Deprecation 경고 무시하지 마
3. 확실하지 않으면 추측하지 말고 docs 읽어

## Docs 구조

```text
node_modules/next/dist/docs/
  index.md          → 전체 개요
  01-app/           → App Router (주력)
  02-pages/         → Pages Router (사용 안 함)
  03-architecture/  → 내부 아키텍처
  04-community/     → 커뮤니티 리소스
```

## 주의

- App Router만 사용. Pages Router 코드 작성 금지.
- `next/dist/docs/01-app/` 가 가장 중요한 레퍼런스.
- 훈련 데이터의 Next.js 패턴을 그대로 쓰지 마 — 반드시 docs 기준으로 작성.

---
> Source: [jinzerokim/fajrjeju](https://github.com/jinzerokim/fajrjeju) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->

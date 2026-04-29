## saju

> 사주팔자(Four Pillars of Destiny) 계산기. pnpm 모노레포로 구성되며,

# .rules — Project Conventions

## Overview

사주팔자(Four Pillars of Destiny) 계산기. pnpm 모노레포로 구성되며,
명세(saju-spec), 핵심 라이브러리(saju-lib), CLI(saju-cli), 계산 앱(saju-app), 가이드 앱(saju-guide)으로 나뉜다.

## Quick Commands

```bash
pnpm install          # 의존성 설치
pnpm build            # 전체 빌드
pnpm build:pages      # Pages 배포용 정적 산출물 조립
pnpm test             # Vitest 테스트 실행
pnpm dev              # saju-app 개발 서버
pnpm dev:guide        # saju-guide 개발 서버
```

## Project Structure

```
packages/
├── saju-spec/        명세/문서/공유 JSON 테이블
├── saju-lib/         핵심 계산 라이브러리 (TypeScript, tsup)
├── saju-cli/         Commander.js CLI (saju-lib 의존)
├── saju-app/         Vite + React 계산 앱 (saju-lib 의존)
│   └── src/
│       ├── App.tsx
│       ├── pages/calculator/
│       └── components/calculator/
└── saju-guide/       Vite + React 가이드 앱 (saju-spec 의존)
    └── src/
        ├── App.tsx
        ├── pages/guide/
        └── components/guide/
```

## Architecture Principles

- **saju-lib is the single source of truth.** 모든 사주 계산 로직은 saju-lib에 집중.
  CLI와 App은 saju-lib의 `calculate()` 함수를 호출하여 결과를 표시만 한다.
- **saju-spec is the documentation/data source of truth.** guide는 `saju-spec`의 문서와 JSON 데이터를 기준으로 설명을 구성한다.
- **No external API dependencies.** 절기 계산(VSOP87), 음양력 변환 모두 자체 구현.
- **Pure functions preferred.** bazi.ts, astro.ts, lunar.ts는 순수 함수로 구성.
- **Pure client-side SPA.** saju-app, saju-guide는 SSR/API Routes 없이 Vite로 정적 빌드.

## Coding Conventions

### Language & Style
- TypeScript strict mode
- 주석/JSDoc: 한국어 중심 (사주 도메인 용어), 영문 병기
- 코드 변수/함수명: 영문 (도메인 용어는 로마자 표기: `yongshin`, `daewon`, `shinsal`)
- No semicolons (prettier default)
- Single quotes for strings

### Type System
- 천간(Stems): `number` 0-9 (甲=0 ~ 癸=9)
- 지지(Branches): `number` 0-11 (子=0 ~ 亥=11)
- 오행(Elements): `'Wood' | 'Fire' | 'Earth' | 'Metal' | 'Water'`
- 성별(Gender): `'Male' | 'Female'`
- 기둥위치(PillarPosition): `'Year' | 'Month' | 'Day' | 'Hour'`

### File Organization (saju-lib)
- `types.ts` — 공유 타입/인터페이스 정의
- `bazi.ts` — 사주 계산 핵심 (기둥, 십성, 합충, 신살, 강약, 용신)
- `astro.ts` — 천문 계산 (절기, 율리우스일)
- `lunar.ts` — 음양력 변환
- `luck.ts` — 대운/세운/월운
- `timezone.ts` — 시간대 처리
- `location.ts` — 위치/LMT 보정
- `i18n.ts` — 한/영 라벨
- `service.ts` — 통합 API (`calculate()`, `parseTime()`)

### Web App (saju-app)
- `src/App.tsx` — 계산 앱 셸
- `src/components/calculator/utils.ts` — 공통 헬퍼 (elementCss, stemSub, branchSub)
- 각 컴포넌트는 `result: SajuResult`와 `i18n: I18n`을 props로 받음
- `vite.config.ts` — `base: '/saju/'` (GitHub Pages 배포 경로)

### Guide App (saju-guide)
- `src/App.tsx` — 가이드 앱 셸
- `src/components/guide/*` — 계산 원리, 표, 규칙 설명 UI
- `vite.config.ts` — `base: '/saju/guide/'`

## Deployment

- GitHub Pages로 자동 배포 (`.github/workflows/deploy.yml`)
- `pnpm build:pages` → `saju-app`은 루트, `saju-guide`는 `/guide/` 경로로 조립
- 배포 URL: `https://luuvish.github.io/saju/`

## Testing

- Vitest 사용, 테스트 파일: `packages/**/__tests__/*.test.ts`, `packages/**/__tests__/**/*.test.ts`
- 코드 변경 후 반드시 `pnpm test` 실행하여 회귀 확인
- `pnpm build`로 TypeScript 컴파일 에러 없음 확인

## Domain Notes

- 일주 경계: 23:00 (자시 구분)
- 음력 변환 범위: 1900-2099
- 대운 기산: 3일 = 1년 비율
- 양남음녀(陽男陰女) 순행, 음남양녀(陰男陽女) 역행
- 절기 기반 월주: 입춘(立春)을 연 시작으로 사용

---
> Source: [luuvish/saju](https://github.com/luuvish/saju) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->

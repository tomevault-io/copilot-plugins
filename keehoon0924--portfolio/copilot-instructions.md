## portfolio

> 원페이지 스크롤 포트폴리오. 디자인 레퍼런스(`design_handoff_portfolio/README.md`)의

# 이기훈 마케터 포트폴리오 — Vite + React + TypeScript

원페이지 스크롤 포트폴리오. 디자인 레퍼런스(`design_handoff_portfolio/README.md`)의
색상·타이포·간격 토큰을 그대로 따른다.

## 스택
- **Vite 6 + React 19 + TypeScript** (이전 Next.js 구현은 git 히스토리에 있음)
- 스타일: 전역 토큰 `src/styles/tokens.css` + 컴포넌트별 **CSS Modules**
- 모션: 외부 라이브러리 없이 `IntersectionObserver` 기반 커스텀 훅 (`src/hooks/`)
- 폰트: Pretendard / IBM Plex Mono / Nanum Pen Script — `index.html`에서 CDN 로드

## 명령
- `npm run dev` — 개발 서버 (5173)
- `npm run build` — 타입체크(`tsc -b`) + 프로덕션 빌드
- `npm run preview` — 빌드 결과 미리보기

## 구조
- `src/components/` 섹션 컴포넌트 (+ `work/` 작업물 카드·목업·오버레이)
- `src/data/` 정적 데이터 (profile, works)
- `src/hooks/` reveal·countup·skillbar·autoscroll·reduced-motion 훅
- `@/` alias → `src/`

## 주의
- 모든 수치(스킬 %, 통계)·연락처·후기는 레퍼런스의 **예시 placeholder**다.
  실제 데이터로 교체 필요 (`// TODO` 주석 표시).
- `prefers-reduced-motion` 존중 — 모든 애니메이션은 비활성화 경로가 있어야 한다.

---
> Source: [keehoon0924/portfolio](https://github.com/keehoon0924/portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->

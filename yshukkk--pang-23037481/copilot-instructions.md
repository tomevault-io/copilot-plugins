## pang-23037481

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 기술 스택

- **React 19** + **TypeScript** + **Vite 8**
- 번들러 플러그인: `@vitejs/plugin-react` (Oxc 기반)
- 린터: **Oxlint** (`.oxlintrc.json`) — ESLint가 아닌 Oxlint를 사용함
- TypeScript는 `tsconfig.json`에서 `tsconfig.app.json`(src용)과 `tsconfig.node.json`(vite.config용)으로 프로젝트 참조 분리

## 주요 명령어

```bash
npm install       # 의존성 설치
npm run dev       # 개발 서버 실행 (http://localhost:5173)
npm run build     # tsc -b (타입 체크) 후 vite build
npm run preview   # 빌드 결과 미리보기
npm run lint      # oxlint 실행
npx tsc -b        # 타입 체크만 단독 실행
```

## 테스트

현재 이 프로젝트에는 테스트 프레임워크가 설정되어 있지 않음 (`package.json`에 test 스크립트 없음). 테스트를 추가할 경우 Vite 생태계와 궁합이 좋은 Vitest 도입을 우선 검토할 것.

## 아키텍처

- 진입점: `index.html` → `src/main.tsx` → `src/App.tsx`
- 현재 `App.tsx`는 최소한의 Hello World 컴포넌트만 렌더링하는 초기 상태이며, 별도의 라우팅/상태관리/스타일링 라이브러리는 아직 도입되지 않음
- `src/index.css`가 전역 스타일 담당 (컴포넌트별 CSS 모듈 등은 아직 없음)

## 기획 문서

이 프로젝트는 팡(Pang) 게임을 구현하는 것이 목표이며, 기획 문서는 `docs/`에 정리되어 있음. 게임 로직 구현 전 반드시 아래 문서를 먼저 확인할 것.

- `docs/PRD.md`: 팡 게임 전반 및 Mission 1 개요
- `docs/FEATURE/main.md`: 메인(타이틀) 화면 구성
- `docs/FEATURE/game_rule.md`: 게임 룰 상세 (플레이어, 풍선 분열, 장애물, 아이템, 승패 조건 등)
- `docs/FEATURE/mission1.md`: Mission 1 난이도 및 세부 규칙
- `docs/PLAN.md`: phase(단계)별 목표를 정리한 문서
- `docs/design/phaseN.md`: 각 phase(N=0, 1, 2, ...)의 상세 설계 문서. `docs/PLAN.md`에 정의된 phase 목표를 구현하기 전 반드시 해당 번호의 설계 문서를 확인할 것. 새 phase 작업 시 동일한 패턴(`docs/design/phase{N}.md`)으로 설계 문서를 추가함

---
> Source: [yshukkk/PANG-23037481](https://github.com/yshukkk/PANG-23037481) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->

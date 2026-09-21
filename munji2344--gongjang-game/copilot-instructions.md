## gongjang-game

> 2D 탑뷰 아이들/타이쿤 게임. Phaser 3 + TypeScript + Vite. itch.io(HTML5) 배포 예정.

# 공장게임

2D 탑뷰 아이들/타이쿤 게임. Phaser 3 + TypeScript + Vite. itch.io(HTML5) 배포 예정.

## 사양서 = Obsidian (유일한 기준, 수시로 변경됨)
경로: C:\Users\User\Desktop\내파일\프로그램 파일\munji2344\공부\코딩\공장게임\
- 공장게임.md — 마스터 기획
- 공장 게임 용어 정리.md — 용어 / 자원
- 공장 게임 CANVAS.canvas — 배치 목업
- Code.md — 로드맵(M0~M7) + 코드 변경 요약 로그 (작업마다 갱신)

## 작업 규칙 (사용자 지정 2026-09-06)
1. 코딩 시작 시 위 Obsidian 폴더 전체 재확인. 변경 있으면 반영.
2. 불명확 / 확인필요 사항은 코드 쓰기 전 전부 질문 (누락 금지).
3. 디자인은 M7에서 일괄 교체 — 지금은 임의 플레이스홀더. 디자인 변경이 게임 로직에 영향 가면 반드시 사전 확인 / 보고.
4. 코드 변경 시 채팅에 상세 설명 + Code.md에 요약 추가.
5. 미확정 수치는 src/data/ 에서만 관리. 현재 전부 임시값 10.

## 구조
- src/data/   — config / resources / recipes / speed / authRules (데이터 표, 하드코딩 금지)
- src/sim/    — grid(타일·설치물), zones(십자 3×3 확장 구역), sim(틱 시뮬·그래프 라우팅·가공·셧다운·배치·구역/해금). Phaser 비의존 순수 TS
- src/auth/   — api (M4 실서버 인증 클라이언트, /api/auth/* 호출)
- src/net/    — save (서버 세이브 로드/저장)
- src/ui/     — screens (메뉴/로그인/회원가입/로딩 DOM 화면 + 흐름)
- src/scenes/ — FactoryScene (렌더 + 입력 + 자동/수동 저장, 플레이스홀더 그래픽)
- src/game.ts — 로그인 통과 후 Phaser 지연 로딩 (세이브 주입)
- server/     — M4 백엔드 (독립 패키지: Express + MongoDB Atlas + bcrypt + JWT). server/.env 는 gitignore
- test/       — vitest (sim + auth 클라이언트). server/test/ 는 서버 자체 vitest
- render.yaml / DEPLOY.md — Render 배포 Blueprint + 배포 절차 가이드 (M4 단계 B)

## 명령
- npm run dev      클라 개발 서버 (5173)
- npm run dev:all  클라 + 서버(3000) 동시 (server/.env 필요)
- npm run build    타입체크 + 빌드
- npm test         클라 로직 테스트  /  npm --prefix server test  서버 테스트

## 현재
- M4 단계 A 완료 (실 인증 + 세이브, 로컬). 단계 B = GitHub 원격 + Render 배포 대기.
- M2 완료 (가공·저장·셧다운, 로컬). 자원/가공품/레시피는 임시 코드(A1/A2/B1/C1, R1/R2) — 용어정리 확정 시 src/data/{resources,recipes}.ts 교체.
- M3 완료 (로컬). A: 십자 3×3 구역 확장 + 칩판매 자원해금. B: 설비 업그레이드(골드+B1)·Node 레벨·Exporter 자동 레벨·tier 판매가. 곡선·배율 전부 임시(config.ts).
- M4 단계 B 준비 완료 (render.yaml + DEPLOY.md). 실행 대기: 사용자가 GitHub 비공개 저장소 생성 → Claude push → 사용자 Render 배포 → Claude .env.production + 빌드 → 사용자 itch.io.
- 세이브 v4 (storage = 자원별 맵). 다음: M4-B 배포 완료 후 M5(랭킹·WAR).

---
> Source: [munji2344/gongjang-game](https://github.com/munji2344/gongjang-game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->

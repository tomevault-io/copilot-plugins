## suddenlab

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

서든랩(SuddenLab)은 React + Cloudflare Workers + D1 기반의 서든어택 커뮤니티 & 분석 플랫폼입니다.
Korean (한국어) UI 기반, 레트로/사이버펑크 테마.

## Repository Structure

- **`/` (root)** — React 19 + Vite 프론트엔드
  - `components/` — React UI 컴포넌트 (Header, CommunityPage, ProfileCard 등)
  - `pages/` — 페이지 컴포넌트
  - `services/` — 외부 API 서비스 (Gemini, Nexon, Updates)
  - `state/` — React Context 상태관리 (AppContext, UIContext)
  - `hooks/` — 커스텀 훅
  - `public/` — 정적 파일
- **`backend/`** — Cloudflare Worker (TypeScript)
  - `src/index.ts` — 메인 라우터 + API 엔드포인트
  - `src/scheduled.ts` — 크론 작업 핸들러
  - `src/routes/` — API 라우트 (updates, reactions, discord)
  - `src/utils/` — 유틸리티 (auth, cors, discord)
- **`bot/`** — Discord 봇 (discord.js)
  - `index.js` — 봇 메인 (메시지 감지 + Ollama 분류)
  - `crawl.js` — Playwright 기반 웹 크롤러
- **`docs/`** — 문서 (git ignored)
- **`logo/`** — 브랜드 에셋

## Tech Stack

| 레이어 | 기술 |
|--------|------|
| 프론트엔드 | React 19 + Vite + TypeScript + Tailwind CSS |
| 백엔드 | Cloudflare Workers + TypeScript |
| 데이터베이스 | D1 (SQLite), Vectorize (1024차원, cosine) |
| 캐시 | Cloudflare KV (채팅 히스토리) |
| AI | Gemini 2.5 Flash, Workers AI (GLM-4.7-Flash), Ollama (gemma3:1b) |
| 디스코드 | discord.js 봇 + N8N 워크플로우 |
| 게임 API | Nexon Open API 프록시 |
| 크롤링 | Playwright (로컬) |
| 배포 | Wrangler (Workers), Vite (프론트) |

## Running the Application

### Frontend dev server
```bash
npm run dev
```
Runs at `http://localhost:3000`.

### Worker dev (local)
```bash
cd backend
npx wrangler dev
```

### Worker deploy
```bash
cd backend
npx wrangler deploy
```

### Discord bot
```bash
cd bot
bash start.sh
```

### N8N (로컬)
```bash
n8n start
```
Runs at `http://localhost:5678`.

## Infrastructure

### Cloudflare Bindings
- **D1:** `sugar-updates` (community_posts, post_reactions, updates, analyses, crawl_logs)
- **Vectorize:** `sugar-updates-index` (1024차원)
- **KV:** `CHAT_HISTORY` (디스코드 멀티턴 대화)
- **Workers AI:** GLM-4.7-Flash (채팅), 제목 필터링
- **Secrets:** DISCORD_BOT_TOKEN, GEMINI_API_KEY, INGEST_API_KEY

### Cron Triggers (KST)
- 08:00 / 20:00 — 미분석 게시물 분석 + 임베딩 생성

### 로컬 서비스 (맥미니 상시 가동)
- **N8N:** 크롤링 워크플로우 + 디스코드 트리거
- **Ollama (gemma3:1b):** 디스코드 메시지 Y/N 분류 전용

## API Endpoints

### Nexon Proxy
- `GET /nexon/*` — 넥슨 API 프록시

### Gemini Proxy
- `POST /gemini/*` — Gemini AI 프록시

### Updates
- `POST /api/updates/ingest` — 크롤링 데이터 저장
- `POST /api/updates/search` — 벡터 검색
- `POST /api/updates/filter` — AI 제목 필터링
- `GET /api/updates/stats` — 통계
- `GET /api/updates/:id` — 단일 조회
- `POST /api/updates/:id/react` — 감정 반응
- `GET /api/updates/:id/reactions` — 반응 조회

### Discord
- `POST /discord/interactions` — 디스코드 인터랙션
- `POST /api/discord/classify` — 메시지 분류
- `POST /api/discord/chat` — AI 채팅 (멀티턴)

### Reactions
- `POST /api/reactions/batch` — 배치 반응 조회

## 감정 반응 시스템

게시물 하단 6개 이모지:
1. 🎉 환호성
2. 👍 좋음
3. 😐 그냥 그럼
4. 🤷 어쩌라고?
5. 👎 싫다
6. 💀 최악이야!

비율 바(얇은 가로 라인)로 반응 비율 표시, 레트로 색상.

## 디스코드 봇 (MumbleCUBE)

### 메시지 처리 흐름
```
유저 채팅 → N8N 감지 → 물음표 포함? → Ollama Y/N 분류 → YES면 Worker AI 응답
```

### 채널 구성
- 불만글 → 비공개 채널로 별도 전송
- 일반 알림 → 공개 채널

## CANNOT 가드레일 (절대 금지)

1. **API 키/토큰을 git에 커밋 금지** — .env, 시크릿은 Cloudflare Secrets + .gitignore
2. **N8N을 외부에 노출 금지** — 로컬 전용, Worker가 중간 역할
3. **디씨 크롤링 욕설 필터 과도 적용 금지** — 디씨 특성상 유의미한 글도 거친 말투
4. **무거운 로컬 모델 사용 금지** — 맥미니 자원 고려, 분류 전용 경량 모델만
5. **구현 전 판단 요청 시 바로 코딩 금지** — 의견 먼저 제시

## 리플레이 데이터 시스템 (진행 중)

- 병영수첩에서 리플레이 JSON 다운로드 자동화
- 전처리 스크립트로 151K → 2.6K 토큰 압축 (98%)
- **대기 중:** 맵별 좌표→위치명 매핑 JSON (유저 제공 예정)

---
> Source: [90cube/SuddenLab](https://github.com/90cube/SuddenLab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->

## agent-hub

> > This file is mirrored across CLAUDE.md, AGENTS.md, and GEMINI.md so the same instructions load in any AI environment.

# Agent Instructions

> This file is mirrored across CLAUDE.md, AGENTS.md, and GEMINI.md so the same instructions load in any AI environment.

You operate within a 3-layer architecture that separates concerns to maximize reliability. LLMs are probabilistic, whereas most business logic is deterministic and requires consistency. This system fixes that mismatch.

## The 3-Layer Architecture

**Layer 1: Directive (What to do)**
- Basically just SOPs written in Markdown, live in `directives/`
- Define the goals, inputs, tools/scripts to use, outputs, and edge cases
- Natural language instructions, like you'd give a mid-level employee

**Layer 2: Orchestration (Decision making)**
- This is you. Your job: intelligent routing.
- Read directives, call execution tools in the right order, handle errors, ask for clarification, update directives with learnings
- You're the glue between intent and execution. E.g you don't try scraping websites yourself—you read `directives/scrape_website.md` and come up with inputs/outputs and then run `execution/scrape_single_site.py`

**Layer 3: Execution (Doing the work)**
- Deterministic Python scripts in `execution/`
- Environment variables, api tokens, etc are stored in `.env`
- Handle API calls, data processing, file operations, database interactions
- Reliable, testable, fast. Use scripts instead of manual work. Commented well.

**Why this works:** if you do everything yourself, errors compound. 90% accuracy per step = 59% success over 5 steps. The solution is push complexity into deterministic code. That way you just focus on decision-making.

## Operating Principles

**1. Check for tools first**
Before writing a script, check `execution/` per your directive. Only create new scripts if none exist.

**2. Self-anneal when things break**
- Read error message and stack trace
- Fix the script and test it again (unless it uses paid tokens/credits/etc—in which case you check w user first)
- Update the directive with what you learned (API limits, timing, edge cases)
- Example: you hit an API rate limit → you then look into API → find a batch endpoint that would fix → rewrite script to accommodate → test → update directive.

**3. Update directives as you learn**
Directives are living documents. When you discover API constraints, better approaches, common errors, or timing expectations—update the directive. But don't create or overwrite directives without asking unless explicitly told to. Directives are your instruction set and must be preserved (and improved upon over time, not extemporaneously used and then discarded).

## Self-annealing loop

Errors are learning opportunities. When something breaks:
1. Fix it
2. Update the tool
3. Test tool, make sure it works
4. Update directive to include new flow
5. System is now stronger

## File Organization

**Deliverables vs Intermediates:**
- **Deliverables**: Google Sheets, Google Slides, or other cloud-based outputs that the user can access
- **Intermediates**: Temporary files needed during processing

**Directory structure:**
- `.tmp/` - All intermediate files (dossiers, scraped data, temp exports). Never commit, always regenerated.
- `execution/` - Python scripts (the deterministic tools)
- `directives/` - SOPs in Markdown (the instruction set)
- `.env` - Environment variables and API keys
- `credentials.json`, `token.json` - Google OAuth credentials (required files, in `.gitignore`)
- `~/.secrets/` - 민감 정보 저장소 (SSH 키, API 키, 서버 접속 정보)

**Key principle:** Local files are only for processing. Deliverables live in cloud services (Google Sheets, Slides, etc.) where the user can access them. Everything in `.tmp/` can be deleted and regenerated.

## Summary

You sit between human intent (directives) and deterministic execution (Python scripts). Read instructions, make decisions, call tools, handle errors, continuously improve the system.

Be pragmatic. Be reliable. Self-anneal.

---

## Automation Management

세션 시작 시 자동화 시스템 상태를 확인합니다.

**자동화 레지스트리**: `directives/automation_registry.md`
- 모든 자동화 요소의 단일 진실 소스
- 상시(always-on) / 요청시(on-demand) / 조건부(conditional) 분류

**폴더 구조**:
```
directives/
├── core/           # 상시 참조 (resource_management 등)
├── workflows/      # 작업별 (deploy, develop_feature 등)
└── agents/         # 복잡한 작업 (pm_agent, sub-agents)
```

**진단 명령어**:
```bash
python execution/automation_manager.py status    # 현황
python execution/automation_manager.py suggest   # 최적화 제안
```

---

## Fallback System (Claude Code → Gemini)

Claude Code 사용량 소진 징후 감지 시:
1. 현재 작업 상태를 `.tmp/fallback_context.json`에 저장
2. `directives/core/fallback_to_gemini.md` 참조
3. Gemini API로 작업 연속성 유지

**전환 트리거**:
- Rate limit 에러 발생
- 사용량 경고 메시지 감지
- 응답 지연 증가

**복귀 조건**:
- Claude Code 사용량 리셋 후 자동 복귀

---

## Secrets Management

민감한 정보는 `~/.secrets/` 폴더에 저장합니다.

**구조**:
```
~/.secrets/
├── hetzner.env        # Hetzner VPS 접속 정보
├── telegram.env       # 텔레그램 봇 설정
├── api_keys.env       # API 키 모음
└── ssh/               # SSH 키 파일들
```

**사용법**:
```bash
# VPS 접속
source ~/.secrets/hetzner.env
ssh $HETZNER_USER@$HETZNER_IP
```

**보안 원칙**:
- 이 폴더는 Git에 절대 커밋하지 않음
- 권한: 700 (본인만 접근 가능)

---

## Infrastructure Overview

### Agent Hub (이 폴더)

**경로**: `/Users/sun/agent-hub/`
**역할**: 모든 프로젝트를 관리하는 중앙 허브
**GitHub**: sun2141/grace-ai

### 연결된 프로젝트 (Project Registry)

| ID | 프로젝트 | 로컬 경로 | GitHub | 배포 |
|----|---------|----------|--------|------|
| palmoni | Palmoni 기도앱 | `/Users/sun/palmoni/` | sun2141/palmoni | palmoni.vercel.app |
| facepick | FacePick | `/Users/sun/facepick/` | - | 개발중 |
| reddit-insight | Reddit Insight | `/Users/sun/reddit-insight/` | - | 개발중 |

### 프로젝트 자동 연결 규칙

**새 프로젝트 추가 시**:
1. 위 테이블에 프로젝트 등록
2. 해당 프로젝트 폴더에 `CLAUDE.md` 생성 (템플릿: `directives/templates/project_claude_md.md`)
3. `directives/projects/{project_id}.md`에 프로젝트별 directive 생성

**Agent가 프로젝트 폴더에서 작업 시**:
- 해당 프로젝트의 `CLAUDE.md` 지침 따름
- 자동화/인프라 작업은 agent-hub에서만 수행
- 프로젝트 간 공유 로직은 `execution/shared/`에 위치

**모니터링 대상 (Hetzner VPS 연동)**:
- 등록된 모든 프로젝트의 배포 상태 감시
- 오류 발생 시 텔레그램 알림 + 자동 수정 시도

### 클라우드 서비스
- **Vercel**: palmoni.vercel.app (프론트엔드 + Serverless API)
- **Supabase**: 데이터베이스 + Auth
- **Google Cloud**: Gemini API, TTS API

### Hetzner VPS (24/7 Agent 서버)
- **IP**: 91.99.58.70
- **접속**: `ssh agent@91.99.58.70`
- **플랜**: CX23 (2 vCPU, 4GB RAM, 40GB NVMe) - €3.99/월
- **OS**: Ubuntu 24.04
- **설치됨**: Node.js 22, Claude Code, PM2, Git

**서버 워크스페이스**:
```
~/workspace/
├── CLAUDE.md              ← 글로벌 Agent 규칙
├── shared-skills/         ← 공유 스킬
├── facepick/              ← FacePick 프로젝트
├── reddit-insight/        ← Reddit Insight 프로젝트
└── prayer-app/            ← Palmoni 연동
```

**핵심 역할**:
- 프로젝트 상시 모니터링
- 오류 자동 감지 및 수정
- 텔레그램 알림 (Bot: 8580472888)
- 스케줄 작업 실행

### 시스템 연결 구조
```
┌─────────────┐     ┌──────────────────────────────┐
│  텔레그램   │◄────│  Hetzner VPS (24/7 Agent)   │
│  (알림/명령) │     │  - 모니터링                  │
└─────────────┘     │  - 오류 자동 수정            │
                    │  - 스케줄 작업               │
                    └──────────────┬───────────────┘
                                   │
        ┌──────────────────────────┼──────────────────────────┐
        │                          │                          │
        ▼                          ▼                          ▼
┌───────────────┐      ┌───────────────┐      ┌───────────────┐
│ Palmoni       │      │ FacePick      │      │ Reddit Insight│
│ (Vercel)      │      │ (개발중)       │      │ (개발중)       │
└───────┬───────┘      └───────────────┘      └───────────────┘
        │
        ▼
┌───────────────┐
│ Supabase      │
│ (DB + Auth)   │
└───────────────┘
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sun2141)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sun2141)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

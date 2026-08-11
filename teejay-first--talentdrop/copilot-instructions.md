## talentdrop

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TalentDrop (职遇) is an AI-powered talent matching platform inspired by Date Drop's deep-matching algorithm. Instead of endless job applications, both candidates and companies complete deep questionnaires (Career DNA / Company DNA), and the algorithm "drops" matched opportunities weekly. The project has a **working full-stack Demo** with dual-side flows, AI chat, resume parsing, knowledge graph visualization, and an interactive theory presentation page.

## Repository Structure

```
vibe-hiring/
├── backend/                    # Python FastAPI backend
│   └── src/
│       ├── main.py             # App entry, router registration
│       ├── seed.py             # Database seed script
│       ├── api/                # REST routes
│       │   ├── auth.py         # JWT authentication
│       │   ├── answers.py      # Questionnaire answer submission
│       │   ├── questions.py    # Question bank retrieval
│       │   ├── company.py      # Company endpoints
│       │   ├── roles.py        # Open position CRUD
│       │   ├── matching.py     # Run matching + dual-action accept/pass
│       │   ├── drop.py         # Weekly drop (candidate + company side)
│       │   ├── scores.py       # DNA score retrieval
│       │   ├── chat.py         # AI conversational profiling
│       │   ├── resume.py       # Resume upload + AI parsing
│       │   ├── profile.py      # User profile CRUD
│       │   └── graph.py        # Knowledge graph + pipeline visualization data
│       ├── models/             # Pydantic models
│       │   ├── database.py     # SQLite schema (all tables)
│       │   ├── user.py, company.py, questionnaire.py, dna_score.py
│       │   ├── match.py        # Dual-action match + drop models
│       │   ├── role.py         # Open position models
│       │   ├── profile.py      # User profile models
│       │   └── chat.py         # Chat message models
│       ├── services/           # Business logic
│       │   ├── scoring.py      # Career DNA scoring
│       │   ├── company_scoring.py, aggregation.py, cas.py
│       │   ├── matching.py     # L1-L2 matching engine (candidate × role)
│       │   ├── drop.py         # Dual drop generation (candidate + company)
│       │   ├── report.py       # Template report (Phase 1)
│       │   ├── llm_report.py   # LLM-powered report (Phase 2)
│       │   ├── chat_service.py # AI chat + entity extraction
│       │   ├── resume_service.py # PDF extraction + AI parsing
│       │   └── graph_service.py  # Knowledge graph data builder
│       ├── data/               # Question banks + seed data
│       └── core/               # Config, deps, logging, middleware
├── frontend/                   # Next.js 16 + React 19 + Tailwind CSS 4
│   └── src/
│       ├── app/
│       │   ├── page.tsx        # Landing / login
│       │   ├── demo/page.tsx   # Interactive theory presentation
│       │   ├── candidate/      # Candidate-side pages
│       │   │   ├── dashboard/, questionnaire/, drop/, chat/, profile/
│       │   │   └── match/[id]/ # Match report + visualization tabs
│       │   └── company/        # Company-side pages
│       │       ├── dashboard/, questionnaire/, candidates/, invite/
│       │       ├── roles/      # Open position management
│       │       ├── drop/       # Company weekly drop
│       │       └── match/[id]/ # Company match detail
│       ├── components/
│       │   ├── layout/         # Header, Sidebar, PageContainer
│       │   ├── ui/             # Button, Card, Input, Modal, Badge, Progress
│       │   ├── questionnaire/  # ProgressHeader, QuestionCard, RankQuestion, BudgetQuestion
│       │   ├── charts/         # RadarChart, KnowledgeGraph, MatchFunnel, DimensionCompare
│       │   └── chat/           # ChatWindow, ChatMessage
│       ├── hooks/              # useAuth, useQuestionnaire
│       └── lib/                # api.ts, types.ts, constants.ts
├── discuss/                    # Product design docs (Chinese)
├── scripts/                    # Dev/deploy shell scripts
└── docker-compose.yml
```

## Key Product Concepts

- **Career DNA**: 8 core dimensions (Pace, Collab, Decision, Expression, Unc, Growth, Motiv, Execution), all spectrums with no "right answer"
- **Weekly Drop**: Tuesday 9PM curated match reveal with compatibility reports
- **Dual Discovery**: Candidates receive matched roles; companies receive matched candidates per role. Both sides Accept/Pass → Mutual Match
- **Three-layer question architecture**: Platform standard (30q, 60%) → Job-type specific (15q, 25%) → Company custom (≤5q, 15%)
- **Matching engine**: L1 hard filter (skills/location) → L2 DNA compatibility (8-dim vector distance × consistency)

## Current Implementation Status

| Feature | Status | Notes |
|---------|--------|-------|
| Career DNA questionnaire | ✅ | 30 questions, 3 question types |
| Company DNA questionnaire | ✅ | 20 questions, CAS scoring |
| L1-L2 matching engine | ✅ | Candidate × Role matching |
| Dual-side Weekly Drop | ✅ | Candidate + Company drops |
| Mutual Match state machine | ✅ | pending → accepted → mutual / passed |
| Open position management | ✅ | CRUD for company roles |
| AI chat profiling | ✅ | OpenAI or mock fallback |
| Resume upload + parsing | ✅ | PDF extraction + AI parse |
| LLM match reports | ✅ | OpenAI or template fallback |
| Knowledge graph visualization | ✅ | Force-directed SVG graph |
| Match pipeline visualization | ✅ | Funnel + dimension compare |
| Theory demo page | ✅ | `/demo` — 6 interactive sections |
| LightRAG integration | ❌ | Dependency installed but not called |

## 本地运行

### 前置依赖

- Python ≥ 3.11 + [uv](https://docs.astral.sh/uv/)
- Node.js ≥ 18 + npm
- （可选）`OPENAI_API_KEY` 写入 `backend/.env`，不配置则 AI 功能自动降级为 mock

### 启动步骤

```bash
# 1. 启动后端（自动创建 .venv 并安装依赖）
bash scripts/dev-backend.sh        # → http://localhost:8000

# 2. 初始化种子数据（另开终端，后端启动后执行一次即可）
bash scripts/seed-db.sh

# 3. 启动前端（另开终端）
bash scripts/dev-frontend.sh       # → http://localhost:3000
```

### 演示账号

统一密码：`demo123`

**候选人**

| 姓名 | 邮箱 | 简介 |
|------|------|------|
| Alex Chen | `alex@example.com` | 高级前端，6年，杭州，快节奏/独立型 |
| Maria Santos | `maria@example.com` | 全栈，4年，北京，协作/使命驱动 |
| James Wright | `james@example.com` | 后端，8年，上海，数据驱动/计划有序 |
| Priya Sharma | `priya@example.com` | 产品工程师，3年，北京，使命驱动/团队 |
| David Kim | `david@example.com` | DevOps，5年，上海，数据驱动 |
| Sophie Zhang | `sophie@example.com` | 初级，1年，南京，广博通才 |

**企业 HR**

| 公司 | 邮箱 |
|------|------|
| Velocity Labs | `hr@velocity-labs.example.com` |
| Meridian Financial | `hr@meridian-financial.example.com` |
| Bloom Education | `hr@bloom-education.example.com` |

> 首页（`http://localhost:3000`）内置了快速登录按钮，点击姓名即可一键登录，无需手动输入。

### 高匹配验证对照

| 候选人 | 公司 | 预期匹配度 |
|--------|------|-----------|
| Alex Chen | Velocity Labs | ~91% |
| James Wright | Meridian Financial | ~92% |
| Priya Sharma | Bloom Education | ~93% |
| Maria Santos | Bloom Education | ~89% |

## Living Documents

- **`discuss/pitch-theoretical-foundation.md`** — Pitch deck with academic backing
- **`discuss/matching-architecture.md`** — Core matching engine architecture

## Design System

- **Colors**: Midnight `#070b1a`, Indigo `#6366f1`, Coral `#ff6b4a`, Amber `#f59e0b`, Emerald `#10b981`
- **Typography**: Syne (display) + DM Sans (body)
- **Style**: Dark theme, glassmorphism cards, frosted glass overlays

## Development Constraints

- **Framework**: Next.js 16+ with React 19+, Tailwind CSS v4, TypeScript (ESM only)
- **File limits**: ≤300 lines per file (JS/TS), ≤8 files per directory level
- **Strong typing**: All data structures must be typed; no `any` without approval
- **Python**: Use `uv` exclusively, virtualenv as `.venv`
- **Docs**: Chinese markdown in `docs/` (formal) and `discuss/` (drafts); all UI text in Chinese
- **Scripts**: Run/debug scripts in `scripts/`, logs to `logs/`
- **Screenshots**: `*.png` 已加入 `.gitignore`，截图不要提交到 git

---
> Source: [Teejay-first/talentdrop](https://github.com/Teejay-first/talentdrop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->

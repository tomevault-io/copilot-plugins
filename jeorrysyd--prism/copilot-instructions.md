## prism

> Prism — Every idea, refracted. An AI-native content creation tool that transforms personal notes into publishable content through a 3-step flow: Signal → Refract → Project.

# Prism (workspace-core) — Claude Code Guide

## Overview

Prism — Every idea, refracted. An AI-native content creation tool that transforms personal notes into publishable content through a 3-step flow: Signal → Refract → Project.
3-step flow: Discover → Angle → Create (Select merged into Discover, Polish merged into Create).
All personalization via `.env` — zero hardcoded values.

## Setup

```bash
cd workspace-core
npm install
cp .env.example .env   # Edit with your settings
npm start              # http://localhost:3456
```

## AI Provider System

Configurable via `AI_PROVIDER` in `.env`:

| Provider | How it works |
|----------|-------------|
| `claude-cli` (default) | Spawns `claude -p` subprocess, uses Claude subscription |
| `anthropic-api` | Uses `@anthropic-ai/sdk` directly, requires `ANTHROPIC_API_KEY` |

Architecture:
```
server/services/
  ai-provider.js          # Factory — reads AI_PROVIDER, exports provider
  providers/
    shared.js             # SSE utilities (startSSE, sendSSE, endSSE)
    claude-cli.js         # Claude CLI provider
    anthropic.js          # Anthropic SDK provider (native streaming)
  storage.js              # File-based project/draft storage
  notes.js                # Markdown notes reader (NOTES_DIR)
```

## .env Variables

```
AI_PROVIDER=claude-cli        # claude-cli | anthropic-api
ANTHROPIC_API_KEY=            # Required if AI_PROVIDER=anthropic-api
PORT=3456
APP_NAME=AI Content Pipeline  # Shown in UI header
OWNER_NAME=User               # Injected into AI prompts
NOTES_DIR=~/path/to/notes     # Markdown notes folder (optional)
FEED_BASE_URL=                # External feed URL prefix (optional, enables feed discovery)
```

## Project Structure

```
workspace-core/
├── .env.example              # Configuration template
├── index.html                # Main shell (single page)
├── css/base.css              # Design system + Pipeline UI styles
├── js/
│   ├── shared.js             # Shared UI utilities (escHtml, formatDate)
│   ├── app.js                # Core: module registry + event bus
│   └── api.js                # HTTP client + SSE streaming
├── modules/
│   └── pipeline/
│       └── index.js          # Pipeline module (3-step UI, project management)
├── server/
│   ├── index.js              # Express entry + /api/config
│   ├── routes/
│   │   └── pipeline.js       # All pipeline API (projects CRUD + 3 steps + sources + drafts)
│   ├── services/
│   │   ├── ai-provider.js    # Provider factory
│   │   ├── providers/        # claude-cli.js, anthropic.js, shared.js
│   │   ├── storage.js        # File-based project/draft storage
│   │   └── notes.js          # Markdown notes service (reads NOTES_DIR)
│   └── skills/               # Skill prompt files (analyze, topics, draft, angle, challenge, polish, headline, adapt)
└── data/                     # Runtime data (gitignored)
    ├── projects/             # Pipeline projects (proj-{uuid}.json)
    ├── drafts/               # Saved drafts (draft-{uuid}.json)
    └── builders/             # Cached builder digests
```

## Pipeline (3 Steps)

### Step 1: Discover
- Scans NOTES_DIR notes by time range
- Fetches external feeds (X/Twitter builders)
- Freeform exploration (drift) and idea tracking (trace)
- Each topic card includes feasibility assessment + suggested direction
- Manual topic entry supported (skip scanning)
- Output: topic candidates with scores, feasibility, direction

### Step 2: Angle
- Generates angle card: hook, stance, evidence, skeleton
- Challenge mode: stress-test your angle
- Reference analysis: paste example content, extract structure
- Output: complete angle card

### Step 3: Create
- Multiple output formats: short-video, short-form, article, academic, pitch
- AI generates content based on angle card
- Inline polish: one-click final draft generation
- Optional 7D quality audit (readability, analogies, logic, quotes, AI-smell, hook, ending)
- One-click polish to final draft
- Headline generation: 10 title candidates with platform optimization
- Cross-platform adaptation: one-click repurpose to other formats
- Save to drafts

## API Endpoints

```
# Projects
GET/POST        /api/pipeline/projects
GET/PUT/DELETE  /api/pipeline/projects/:id

# Pipeline Steps (all SSE streaming)
POST  /api/pipeline/discover       # Step 1: topic discovery (notes/feed/drift/trace)

POST  /api/pipeline/angle          # Step 2: angle card design
POST  /api/pipeline/angle/challenge  # Step 2: stress-test angle
POST  /api/pipeline/angle/reference  # Step 2: extract structure from example
POST  /api/pipeline/create         # Step 3: generate content
POST  /api/pipeline/polish         # Step 3: inline polish (review/final)
POST  /api/pipeline/headline       # Step 3: generate title candidates
POST  /api/pipeline/adapt          # Step 3: cross-platform adaptation

# Sources
GET   /api/pipeline/sources/notes
GET   /api/pipeline/sources/feed

# Drafts
GET/POST        /api/pipeline/drafts
PUT/DELETE      /api/pipeline/drafts/:id
```

## Frontend Architecture

- **No framework** — vanilla JS with IIFE module pattern
- **Event delegation** — click handlers use `data-action` attributes
- **Single module** — pipeline/index.js handles all UI
- **Shared utilities** — `js/shared.js` provides escHtml, formatDate
- **SSE streaming** — `js/api.js` provides stream() for real-time AI output

## Skill Routing

When the user's request matches an available skill, ALWAYS invoke it using the Skill
tool as your FIRST action. Do NOT answer directly, do NOT use other tools first.

Key routing rules:
- Product ideas, brainstorming → invoke office-hours
- Bugs, errors, "why is this broken" → invoke investigate
- Ship, deploy, create PR → invoke ship
- QA, test the site → invoke qa
- Code review → invoke review
- Architecture review → invoke plan-eng-review

---
> Source: [Jeorrysyd/Prism](https://github.com/Jeorrysyd/Prism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->

## claudeboost

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## IMPORTANT: Auto-Boost All Prompts

**YOU MUST call the `boost_prompt` MCP tool on EVERY user task prompt before executing it.** This is not optional. Follow the EXACT same UI format as the `/boost` skill every time:

1. Call `boost_prompt` with the user's prompt
2. Display the result using THIS format (always the same — unified UI):
   ```
   ⚡ **CLAUDEBOOST** · `{domain}` · Level: `{level}`
   ---
   > 📝 **Original:** {original}
   ---
   ### ✨ Boosted Prompt
   {complete boosted text}
   ---
   ```
3. Present the choice modal via AskUserQuestion (Use boosted / Add notes & refine / Keep original)
4. Execute whichever version the user chooses

**Authentication:** If `boost_prompt` returns `{"error": "auth_required"}`, show the auth message and open the browser to `https://claudeboost.vercel.app/auth/cli-login`. Do NOT fall back to manual enhancement. Wait for the user to authenticate and try again.

**Exceptions — skip the boost when:**
- The user ends their prompt with `--raw` (remove the suffix, execute directly)
- The `auto_boost` setting is `false` (check via `boost_settings` tool with `{"action": "get"}`)

Examples:
- `build me an API endpoint` → BOOST this (call boost_prompt, show comparison, ask)
- `build me an API endpoint --raw` → DO NOT boost, execute directly
- `fix the login bug --raw` → DO NOT boost, execute directly
- `analyze churn data` → BOOST this

**Do NOT boost these types of messages:**
- Questions about the codebase ("how does X work?", "what does Y do?")
- Git operations ("commit this", "push to main")
- Simple file reads ("show me server.py")
- Conversational replies ("yes", "no", "looks good", "try again")
- Messages that are clearly responses to your questions, not new tasks

**DO boost these — any prompt that is a task/instruction:**
- "build X", "create X", "implement X", "add X"
- "fix X", "debug X", "refactor X"
- "analyze X", "write X", "design X"
- "set up X", "configure X", "deploy X"

---

## Project Overview

ClaudeBoost is a Claude Code MCP plugin that enhances user prompts before sending them to Claude. Two parts:

1. **MCP Server** (Python, `mcp-server/`) — registers a `boost_prompt` tool that classifies prompt domain, rewrites the prompt using Claude API with enterprise playbook rules, and logs results. Uses Haiku for classification, Sonnet for enhancement.
2. **Web Dashboard** (Next.js 14, `web-dashboard/`) — displays boost history, ratings/feedback, score histograms, ROI metrics, and per-domain constraint management. Reads live data from `~/.claudeboost/` with 5s auto-polling.

Documentation: `.claude/docs/README.md` (index of all docs)

## Commands

### MCP Server
```bash
cd mcp-server
pip3 install -r requirements.txt
python3 -m pytest tests/ -v          # run all tests (25 tests)
python3 -m pytest tests/test_feedback.py -v   # run single test file
```

### Web Dashboard
```bash
cd web-dashboard
npm install            # install dependencies
npm run dev            # dev server at claudeboost.vercel.app
npm run build          # production build
npm run lint           # ESLint
npm run test:watch     # Vitest (watch mode)
```

## Architecture

### MCP Server (`mcp-server/`)
- `server.py` — MCP entry point, registers `boost_prompt` tool, orchestrates flow
- `classifier.py` — Classifies prompts into 7 domains via Claude Haiku API
- `enhancer.py` — Enhances prompts with enterprise playbook rules via Claude Sonnet API
- `feedback.py` — Reads/writes `~/.claudeboost/history.json` and `config.json`

Dependency order: `feedback.py` → `classifier.py` → `enhancer.py` → `server.py`

### Web Dashboard (`src/`)
- Vite 5 + React 18 + TypeScript + Tailwind CSS 3 + shadcn/ui
- React Router v6: History (`/`), Stats (`/stats`), Constraints (`/constraints`)
- Path alias: `@` maps to `./src`

### Domain Model
7 domains: `data_science`, `data_engineering`, `business_analytics`, `general_coding`, `documentation`, `devops`, `other`

### Data Storage
Local JSON files at `~/.claudeboost/`: `history.json` (boost log) and `config.json` (per-domain constraints)

## Skills

- `/boost <prompt>` — manually boost a specific prompt with full UI flow
- `/boost-settings` — view current settings
- `/boost-settings --level <light|medium|full>` or `-l` — change boost intensity
- `/boost-settings --auto <true|false>` or `-a` — toggle auto-boost
- `/boost-help` — show all commands and usage guide
- `--raw` suffix — skip auto-boost for a single prompt

---
> Source: [FarhadS08/claudeboost](https://github.com/FarhadS08/claudeboost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->

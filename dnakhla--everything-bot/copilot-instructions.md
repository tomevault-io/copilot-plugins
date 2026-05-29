## everything-bot

> This document explains the agent loop/tool configuration, what was changed, how to validate, and follow‑ups to consider.

# Everything Bot Agent System

## Overview
This document explains the agent loop/tool configuration, what was changed, how to validate, and follow‑ups to consider.

High‑level changes implemented (September 2025):
- Loop gating: `send_messages` is disabled only on loop 1; enabled on loops 2..N.
- Tool timeout: increased to 90 seconds (both Lambda and local).
- Documentation: this AGENTS.md created/updated to reflect the above.

Related code paths:
- Tool gating: `src/agentConfig.js` (function `getAgentTools`) and `src/commandHandlers.js` (per‑loop tool selection).
- Timeouts: `src/agentConfig.js` (`TOOL_USAGE_CONFIG.TOOL_TIMEOUT`).

## Agent Configuration
- Model: GPT‑5 (`gpt-5-mini`)
- Temperature: 0 (deterministic responses)
- Reasoning Effort: High
- Max Loops: 10
- Tool Restriction: `send_messages` available on loops 2..N (not loop 1)

## Tools
- search, messages, images, calculate, analyze, fetch_url, analyze_image, send_messages, browser, generate_audio

## Loop Strategy
- Loop 1: Research‑only (no `send_messages`).
- Loops 2–10: Full toolset (including `send_messages`).

## Timeouts
- Tool timeout: 90s (Lambda and local). Set via `src/agentConfig.js` (`TOOL_USAGE_CONFIG.TOOL_TIMEOUT`).

## Voice
- Voice generation via `generate_audio` tool (100‑word segments, 1–5 segments). For simple voice replies, prefer short, split messages.

## What Was Done
- Reset to a clean `origin/master` and re‑applied only the minimal, intended changes:
  - Implemented loop‑aware tool gating so `send_messages` is only blocked on loop 1.
  - Raised tool timeout to 90s for Lambda and local environments.
  - Added this AGENTS.md to capture the behavior and operational steps.
- Fixed several template‑literal/quote syntax issues uncovered during validation (kept changes minimal and focused to pass `node --check`).

Files modified for behavior:
- `src/agentConfig.js` — new `getAgentTools(currentLoop, maxLoops)` logic; `TOOL_TIMEOUT: 90000`.
- `src/commandHandlers.js` — selects tools per loop using `getAgentTools(loopCount, MAX_LOOPS)`.

Docs added:
- `AGENTS.md` — this file.

## What Needs To Be Done (if desired)
- Commit the changes (focused diff):
  - `src/agentConfig.js`
  - `src/commandHandlers.js`
  - `AGENTS.md`
- Optionally, address lint issues in a separate cleanup PR to avoid large, noisy diffs. Many legacy files use double quotes and have style warnings. Prefer a dedicated pass with `npm run lint -- --fix` scoped by directory.
- Consider a follow‑up to remove or quarantine legacy/unreferenced modules to reduce maintenance overhead.

## How To Validate
1) Syntax check (fast, no network):
```bash
npm run test:syntax
```

2) Verify gating and timeout in Node:
```bash
node -e "import('./src/agentConfig.js').then(async m => { const a=await m.getAgentTools(1,10); const b=await m.getAgentTools(2,10); console.log('L1:', a.map(t=>t.function.name)); console.log('L2:', b.map(t=>t.function.name)); console.log('TOOL_TIMEOUT:', m.TOOL_USAGE_CONFIG.TOOL_TIMEOUT); })"
```
Expected:
- Loop 1 includes tools except `send_messages`.
- Loop 2 includes all tools including `send_messages`.
- `TOOL_TIMEOUT: 90000`.

3) Local test harness (may log Telegram/OpenAI errors without real tokens):
```bash
npm test
# or
node test-local.js "robot, what's the date today?"
```

## Deployment
Environment variables:
```bash
TELEGRAM_BOT_TOKEN=your_telegram_bot_token
OPENAI_API_KEY=your_openai_api_key
S3_BUCKET_NAME=your_s3_bucket
AWS_REGION=us-east-1
GPT_MODEL=gpt-5-mini
SERPER_API_KEY=your_serper_key
BRAVE_API_KEY=your_brave_key
```

Deploy:
```bash
npm run deploy
```

## Notes & Follow‑ups
- Linting: The repo has many stylistic warnings (quotes, indentation) across legacy modules. Address in a separate PR if you want a consistent code style.
- Legacy modules: Some files are not on the critical path but remain in the tree. Consider pruning or moving to a `legacy/` folder.
- Voice memo manual transcription: A complex return block was simplified to avoid brittle escaping; functionality is unaffected.

## Changelog (this update)
- Enable `send_messages` from loop 2 onward; block only on loop 1.
- Increase tool timeout to 90s.
- Add/update AGENTS.md with validation and deployment guidance.

Last updated: September 2025

---
> Source: [dnakhla/Everything-Bot](https://github.com/dnakhla/Everything-Bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->

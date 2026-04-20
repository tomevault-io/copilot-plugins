## agent-orchestrator

> Sequential pipeline of Claude agents for automated software development. An optional Spec Discovery agent refines ideas through conversation, then the task flows through pipeline agents: Product Designer, Test Writer, Frontend Builder, Backend Builder, Code Reviewer, PM, Git Committer. The output is a fully runnable application in `workspace/<run-id>/project/`.

# Agent Orchestrator

## Project Overview
Sequential pipeline of Claude agents for automated software development. An optional Spec Discovery agent refines ideas through conversation, then the task flows through pipeline agents: Product Designer, Test Writer, Frontend Builder, Backend Builder, Code Reviewer, PM, Git Committer. The output is a fully runnable application in `workspace/<run-id>/project/`.

## Architecture
- `orchestrate.js` — Main pipeline entry point, ES module, uses `claude --print` CLI
- `discover.js` — Interactive spec discovery CLI (multi-turn conversation, optional)
- `agents/prompts/*.md` — System prompts for each agent (8 files)
- `workspace/<run-id>/` — Output artifacts per run (gitignored)
- `workspace/<run-id>/project/` — Extracted runnable project files (package.json, src/, tests/)
- `logs/<run-id>.log` — Full pipeline logs (gitignored)

## Key Patterns
- Each agent gets isolated context (only the inputs it needs)
- Builder agents output fenced code blocks with file paths; `extractCodeFiles()` parses these and writes real files to `project/`
- Code Reviewer loops up to `MAX_REVIEW_ITERATIONS` (3) times, feeding feedback back to builders
- Two model tiers: `claude-opus-4-6` for complex work, `claude-sonnet-4-6` for simpler tasks
- Backend serves frontend as static files — single `npm start` runs everything
- `npm install` runs automatically after code extraction

## Running
```bash
node orchestrate.js                    # interactive spec/task selector
node orchestrate.js "task description" # direct orchestration
node orchestrate.js --resume           # resume a previous run from any step
node discover.js --run                 # interactive spec discovery, then orchestrate
```
Requires Claude Code CLI (`claude`) to be installed and authenticated. All agent calls use `claude --print` — no separate API key needed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hanssonfredrik) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

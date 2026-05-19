## pi-xai-oauth

> > **For AI coding agents only.** Keep this file machine-readable and concise. Human-facing docs live in README.md.

# AGENTS.md — AI Agent Operations Manual for pi-xai-oauth

> **For AI coding agents only.** Keep this file machine-readable and concise. Human-facing docs live in README.md.

## Project Overview
pi-xai-oauth is a pi-package that registers the xAI OAuth provider ("xai-auth") and Grok models (including grok-4.3 with 1M context + reasoning) for the pi coding agent framework.

Core flow: `bin/setup.js` → `pi install` → provider registration in `extensions/xai-oauth.ts` → OAuth PKCE login → streaming via xAI API.

## Key Commands (Exact, Copy-Paste Ready)
- Install / setup: `node bin/setup.js` or `npm run setup` (if added)
- Install as pi extension: `pi install npm:pi-xai-oauth`
- Run TypeScript: `npx tsc --noEmit` (validate)
- Git: Always work on feature branches. Current branch for this work: `feature/improved-agent-scaffolding`

## Architecture & Boundaries (MUST / MUST NOT)
**MUST:**
- Register providers via `pi.registerProvider("xai-auth", { ... })`
- Use PKCE OAuth flow with local callback server
- Support reasoning levels: none / low / medium / high
- Reuse `~/.grok/auth.json` when possible
- Keep models list in sync with xAI releases

**MUST NOT:**
- Hardcode API keys (use OAuth only)
- Modify core pi-coding-agent internals
- Touch unrelated extensions or skills
- Skip error handling on OAuth refresh

## File Structure & Wayfinding
```
pi-xai-oauth/
├── bin/
│   └── setup.js          # One-command installer + settings seeder
├── extensions/
│   └── xai-oauth.ts      # Core provider registration + OAuth logic (start here for changes)
├── package.json
├── tsconfig.json
├── README.md
├── AGENTS.md             # This file
└── .scaffold/            # Persistent agent state (auto-generated on init)
    ├── plan.md
    ├── constraints.md
    ├── progress.md
    ├── context.md
    └── (custom overrides here)
```

Start any task by reading:
1. `extensions/xai-oauth.ts` (lines 600+ for registerProvider)
2. `bin/setup.js`
3. This AGENTS.md

## Style & Quality Rules
- Use TypeScript strict mode
- Prefer async/await for OAuth and API calls
- Add JSDoc for all exported functions
- Keep OAuth callback server minimal and secure
- Never log sensitive tokens

## Safety Gates
- Before any file edit: run `git status` and confirm on correct branch
- Before committing: ensure `npx tsc --noEmit` passes
- For multi-agent work: always use the subagent tool with explicit parallel or chain mode
- External state lives in `.scaffold/` — update progress.md after every major step

## Multi-Agent Workflow (Preferred)
When complex work is needed:
1. Use `subagent` in PARALLEL mode for research + planning
2. Delegate to specialized agents (researcher, planner, reviewer, worker)
3. Save outputs to `.scaffold/` files
4. Review with `reviewer` agent before implementation

## Persistent State (Use These Files)
- `.scaffold/plan.md` — Current implementation plan with steps and owners
- `.scaffold/constraints.md` — Hard rules and boundaries
- `.scaffold/progress.md` — What has been done + next actions
- `.scaffold/context.md` — Shared context for handoff between agents

## Next Steps When Starting Fresh
1. Read this AGENTS.md + README.md
2. Run `git checkout -b feature/your-task`
3. Check `.scaffold/plan.md` for current work
4. Use parallel subagents for heavy lifting

This file should be updated whenever architecture, commands, or rules change.

---
> Source: [BlockedPath/pi-xai-oauth](https://github.com/BlockedPath/pi-xai-oauth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->

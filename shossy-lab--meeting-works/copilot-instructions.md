## meeting-works

> 0. Git sync: `git fetch` — if behind remote, warn user and recommend `git pull`

# Meeting-Works — Durable Instructions

@import README.md

## Session Protocol

### Start (MANDATORY — every session)
0. Git sync: `git fetch` — if behind remote, warn user and recommend `git pull`
1. Read INTENT.md and CODEBASE_MAP.md
2. Read latest session state doc in docs/sessions/ — check staleness (warn if >7 days)
3. Three-Criteria Agent Evaluation (see .claude/rules/autonomous-agents.md)
4. Load contracts for features being touched
5. Report ready state to user

### During Session (INCREMENTAL WRITES — non-negotiable)
- Write to session log (docs/sessions/{YYYY-MM-DD}.md) after each significant action — NEVER batch to session end
- Log feature gaps/friction points to docs/features/backlog.md IMMEDIATELY when encountered
- Update contracts when boundary code changes — in the same edit sequence
- Update docs when referenced structures change — before moving to the next task
- Use /compact at natural breakpoints; after compaction, re-read CLAUDE.md + INTENT.md + CODEBASE_MAP.md + latest session log

### Finish (MANDATORY — before every session end)
You MUST execute this protocol before indicating task completion. NEVER say "let me know if you need anything else" without first completing these steps:
1. Verify build passes (language-appropriate build/type-check/lint)
2. Spawn 2-agent finish team (both Opus):
   - Agent 1: Update CODEBASE_MAP.md (file registry, dependency graph, recent changes)
   - Agent 2: Curate INTENT.md (new decisions, reasoning, trade-offs)
3. Validate agent output (check git diff scope, verify quality)
4. Sync contracts — verify all contracts referenced by changed code are current
5. Finalize session log (should be ~90% written from incremental updates)
6. Create/update session state doc (what was done, decisions, issues, next steps)
7. Review feature backlog entries logged during session
8. Stage and commit atomically (code + docs + contracts together)
9. Report to user (accomplished, what agents updated, next steps)
10. Ask before pushing — never auto-push

## Context Budget

| Layer | Files | When | Target |
|-------|-------|------|--------|
| L0 | CLAUDE.md + imports | Always | <100 lines + imports |
| L1 | INTENT.md + CODEBASE_MAP.md | Every session | <200 lines each |
| L2 | Contracts + session log | Per-task | <1KB per contract |
| L3 | Feature docs, specs | On-demand | No limit |

Never load L3 files unless actively implementing from them. Use subagents for research to keep main context clean.

## Code Conventions
- TypeScript strict mode, no `any` — use `unknown` and narrow, or define a proper type
- Electron (main process) + React (renderer) for the desktop app
- Vercel API Routes (Node.js) for backend — transcription orchestration, auth, data access
- Supabase for database (PostgreSQL), auth (email/password, magic link), and file storage (audio, voice refs)
- Supabase queries in API routes, not client-side — never import the browser client in a server context
- OpenAI API (`gpt-4o-transcribe-diarize`) for transcription with speaker diarization
- Claude API for meeting analysis (summaries, action item extraction)
- Zod for runtime validation at API boundaries
- Tailwind for styling — no CSS modules unless wrapping third-party components
- Conventional commits: type(scope): description
- Named exports only — no default exports (except where framework requires)
- Use @/ path alias for all imports; never use `../../../`
- Co-locate related files in feature directories
- Use existing project patterns before introducing new ones

## Rules
1. Follow session protocol every time. No exceptions.
2. Write session logs incrementally — never batch to session end.
3. Keep always-loaded files within context budget targets.
4. Update contracts and docs when the code they reference changes — same commit.
5. Ask before pushing to remote.
6. When in doubt, read existing code first.
7. Log feature gaps and friction points immediately.
8. All agents must use Opus model.

## Claude Code Standardization

This project follows the standardized Claude Code setup defined in `_claude-setup` (Shossy-lab/_claude-setup). The standard includes:
- **Hook-enforced memory** — PreCompact/PostCompact hooks auto-persist session state across compactions
- **Contract enforcement** — PostToolUse hooks remind about contract sync after every file edit
- **Session protocols** — /start-session and /finish-session slash commands with autonomous doc swarm
- **Agent library** — 7 standard agents (code-reviewer, db-specialist, ui-specialist, doc-updater, contract-auditor, codebase-mapper, intent-curator)
- **Branch protection** — PreToolUse hook blocks checkout/push to main

### Skills Setup
Mount the shared skills repo (if not already linked):
```bash
ln -s A:/claude-skills .claude/skills
```
Skills provide: deep-planning, swarm, plan-execute, batch-write, commit-batch, validate-kb, worktree-sync, branding, index-docs, onboard, and more.

### Onboarding
To bring this project up to the full standard setup, run `/onboard` which deploys the _claude-setup template and populates project-specific data.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Shossy-lab) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->

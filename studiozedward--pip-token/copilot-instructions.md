## pip-token

> A VS Code extension that helps Claude Code users track their token usage with a Fallout Pip-Boy aesthetic.

# Pip-Token — Project Conventions

A VS Code extension that helps Claude Code users track their token usage with a Fallout Pip-Boy aesthetic.

## Tech stack

TypeScript, VS Code Extension API, SQLite (better-sqlite3), plain HTML/CSS in the webview. No frontend frameworks. No chart libraries. No network calls.

## Commands

- `npm install` — install dependencies
- `npm run build` — compile and bundle with esbuild
- `npm run watch` — dev mode
- `npm test` — run Vitest tests
- `npm run package` — build .vsix
- F5 in VS Code — launch the extension in a development host

## Branching and commits

- `main` is always shippable
- Feature branches off main, named `feat/short-description` or `fix/short-description`
- Squash-merge into main via PR
- Conventional Commits: `feat: ...`, `fix: ...`, `docs: ...`, `refactor: ...`, `test: ...`, `chore: ...`

## Where things live

- Product spec: `docs/DESIGN.md` (authoritative)
- Screenshots: `mockups/screenshots/*.png` (current state of the extension)
- Terminology: `docs/GLOSSARY.md`
- Architecture: `docs/ARCHITECTURE.md`
- Decisions made and why: `docs/decisions/`
- Lessons learned during build: `LESSONS.md` — **read this before tackling areas marked as "previously problematic"**

## Subdirectory CLAUDE.md files

Pip-Token uses a distributed CLAUDE.md pattern. When working in `src/parsing/`, `src/webview/`, `src/data/`, `src/domain/`, or `test/`, also read the local `CLAUDE.md` in that directory — it contains conventions specific to that subsystem.

## Session identity

A "session" in Pip-Token maps 1:1 to a Claude Code `sessionId`. Clearing context in Claude Code starts a new session with a new ID — this is intentional, not a bug. Context window utilisation and cache state are per-session and cannot be meaningfully merged across sessions. The "All Projects" aggregate on LIVE/SESSION sums token counts from the turns table across all active sessions (not from window counters, which break across rate-limit boundaries and resyncs).

## Hard rules

- **v1 makes no network calls. Anywhere.** Don't add any network code, even libraries that imply network capability. Telemetry and update checks are strictly v2+ territory and would need their own ADR superseding 0004.
- All cost math reads from `src/domain/pricing.json`. Never hardcode prices.
- Peak hour schedule lives only in `src/domain/peakHourSchedule.ts`. Never duplicate.
- Honest uncertainty: use the LEARNING state, not zeros or fake projections.
- Errors surface as subtle in-panel Pip-Boy-themed warnings (e.g., "PARSER: 3 LINES SKIPPED"), never VS Code toast notifications.

## Update LESSONS.md when

You spend more than 30 minutes tracking down a non-obvious bug, or you discover a platform/library quirk worth remembering. See `LESSONS.md` for the format.

---
> Source: [studiozedward/pip-token](https://github.com/studiozedward/pip-token) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->

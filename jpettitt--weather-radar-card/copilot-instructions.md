## weather-radar-card

> The canonical rules for AI coding agents working on this repository

# CLAUDE.md

The canonical rules for AI coding agents working on this repository
live in [`AGENTS.md`](AGENTS.md). Read it first. Everything below is
Claude-specific supplement, not override.

## Claude-specific notes

- Per-user Claude Code memory under `~/.claude/projects/<project>/`
  is your working knowledge — saved context from prior sessions
  (user preferences, decisions, project state). It complements
  [`AGENTS.md`](AGENTS.md); it does **not** override it. If a
  memory entry conflicts with [`AGENTS.md`](AGENTS.md),
  [`AGENTS.md`](AGENTS.md) wins and the memory entry should be
  updated.
- The user's global instructions (their personal `~/.claude/CLAUDE.md`)
  may impose additional rules — most commonly around when not to
  push or commit without explicit approval. Respect those on top of
  the project rules. The rule "Always allow the user to test code
  before pushing it" survives any session-level commit-autonomy
  grant.
- The **diagnostic discipline** in [`AGENTS.md`](AGENTS.md) ("no
  fixes without understanding") is load-bearing. It exists because
  the rule was learned from real incidents in this codebase — the
  `leaflet.markercluster` init race ([#110](https://github.com/jpettitt/weather-radar-card/issues/110))
  is the canonical example. Treat it as a hard constraint, not
  advisory.
- `main` is the canonical and only long-lived branch. The repo had a
  `master` branch up to mid-2026 from its pre-transfer history; it
  has been removed. If a memory entry says otherwise, update it.

---
> Source: [jpettitt/weather-radar-card](https://github.com/jpettitt/weather-radar-card) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->

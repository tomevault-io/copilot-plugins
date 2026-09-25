## nudge

> This repository's agent instructions live in `AGENTS.md`, with topic-specific

# Nudge — Claude Code instructions

This repository's agent instructions live in `AGENTS.md`, with topic-specific
guidance under `.agents/`. Claude Code imports them here so there is one source
of truth shared by every agent tool.

@AGENTS.md

## Claude Code specifics

- Read `.agents/swift-swiftui.md` before changing Swift code or the UI.
- Read `.agents/simulation-testing.md` for configuration, SOFA, deadline, and
  startup work. Run the relevant command-line simulations and verify their
  unified logs, using isolated fixtures and the existing simulated clock.
- Follow the environment-neutral coordination guidance in `AGENTS.md`; Claude
  agent teams are optional and must not block authorized work.
- Never commit or push unless asked. Work on a branch if on `main`.

---
> Source: [macadmins/nudge](https://github.com/macadmins/nudge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->

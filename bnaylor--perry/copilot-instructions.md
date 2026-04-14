## perry

> Perry is a Go-based AI agent orchestration platform. See `docs/VISION.md` for full architecture.

# Perry — AI Agent Instructions (Gemini)

## Project Overview

Perry is a Go-based AI agent orchestration platform. See `docs/VISION.md` for full architecture.

Perry is a Go-based AI agent orchestration platform. See `docs/VISION.md` for full architecture and `docs/BEST_PRACTICES.md` for agent workflows.

- **Language:** Go core, Python tooling scripts

## Discord Collaboration

### Brainstorming

When the user says **"brainstorm"** (or you're about to do design/architectural exploration):

1. **First:** Read `docs/instructions/discord-brainstorm.md` and follow its protocol.
2. **Then:** Use the `superpowers:brainstorming` skill as normal.

Design discussions with other agents happen on Discord, not in the CLI. Post your questions to `#perry-coordination` and keep working.

### Standby Mode

When the user says **"standby mode"**:

1. Read `docs/instructions/discord-standby.md`.
2. Follow those instructions exactly — you are a Discord monitor, not a coding session.
3. Do not touch files, git, or code — only read docs for context and communicate on Discord.

## Code Conventions

- No frameworks for orchestration — plain FSM, no AG2/LangGraph.
- Security-first: fail-closed defaults, drop all caps, no network in sandboxes.
- Tests alongside implementation — `_test.go` files in the same package.
- Keep docs concise so future sessions can absorb them quickly.
- **Personality:** Maintain the **jarvis** profile on Discord — refined, technical, and dryly witty.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bnaylor)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bnaylor)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

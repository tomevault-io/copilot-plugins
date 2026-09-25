## screen-builder-poc

> This repo uses [AGENTS.md](./AGENTS.md) as the top-level entrypoint.

# Agent Instructions (Gemini)

This repo uses [AGENTS.md](./AGENTS.md) as the top-level entrypoint.

For the long-form protocol, see [docs/knowledge-base/AGENTS.md](docs/knowledge-base/AGENTS.md).

This repo uses a curated Gemini command layer, not a full mirror of `.agent/workflows/`. The shared source of truth remains `AGENTS.md`, `.agent/`, and the knowledge-base docs.

Available project Gemini commands:
- `/screen:onboard`
- `/screen:execute-plan`
- `/screen:wrap-up`

After modifying files in `.gemini/commands/`, run `/commands reload` in Gemini CLI to refresh the command list without restarting.

For faster onboarding in a fresh Gemini session, also read:

- [CONTRIBUTING.md](./CONTRIBUTING.md)
- [docs/knowledge-base/ARCHITECTURE.md](./docs/knowledge-base/ARCHITECTURE.md)
- [docs/knowledge-base/STATE-MODEL.md](./docs/knowledge-base/STATE-MODEL.md)
- [docs/working/README.md](./docs/working/README.md)

---

*This file routes Gemini into the same Screen Studio source of truth as OpenAI Codex, Claude, and GitHub Copilot.*

---
> Source: [dthompson-jti/screen-builder-poc](https://github.com/dthompson-jti/screen-builder-poc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->

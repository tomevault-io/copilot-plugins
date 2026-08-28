## dsh-goalmesh

> This repository contains a DeepSeek Harness Plugin, not a Codex Plugin.

# DSH GoalMesh

This repository contains a DeepSeek Harness Plugin, not a Codex Plugin.

- Treat the architecture in `docs/architecture.md` as the implementation contract.
- Keep the installable bundle, Host tool, and Web companion as separate packages.
- Preserve Cordis lifecycle ownership: every registration and live resource must be disposed with its owning fiber.
- Run `pnpm check` before each implementation-plan commit.
- Keep one commit per numbered point in `docs/execution-plan.md`.

---
> Source: [Jarad-z/dsh-goalmesh](https://github.com/Jarad-z/dsh-goalmesh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->

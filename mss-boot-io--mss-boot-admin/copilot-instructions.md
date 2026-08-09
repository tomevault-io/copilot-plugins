## mss-boot-admin

> Use the repository Agent contract and MSS machine-readable project context for every task.


# MSS Agent-native foundation

Treat `AGENTS.md` and `.mss/` as the project source of truth.

Before editing:

1. Read the root `AGENTS.md` and the closest nested `AGENTS.md`.
2. Inspect `.mss/project.yaml`, `.mss/capabilities.yaml`, and `.mss/commands.yaml`.
3. Run `go run ./cmd/mss context --format json` when architecture or existing capabilities are unclear.
4. Reuse a workflow from `.agents/skills/` when it matches the request.
5. Use structured module/application specifications and deterministic generators before writing repetitive code by hand.
6. Run `go run ./cmd/mss verify --changed` before declaring completion.

Never invent workstation-specific absolute paths, production credentials, undocumented repository commands, or a parallel source of project truth. Keep this rule thin; detailed policy belongs in `AGENTS.md`, `.mss/`, Skills, tests, and ADRs.

---
> Source: [mss-boot-io/mss-boot-admin](https://github.com/mss-boot-io/mss-boot-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->

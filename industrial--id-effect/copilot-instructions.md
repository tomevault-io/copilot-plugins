## roam

> This project includes **roam** ([roam-code](https://github.com/Cranot/roam-code)) via **devenv** for indexed codebase comprehension. Prefer roam for cross-cutting exploration, blast radius, and structured context; use normal search, grep, and file reads when you already know the path or need a quick string hit.


## Codebase navigation (`roam`)

This project includes **roam** ([roam-code](https://github.com/Cranot/roam-code)) via **devenv** for indexed codebase comprehension. Prefer roam for cross-cutting exploration, blast radius, and structured context; use normal search, grep, and file reads when you already know the path or need a quick string hit.

**Index:** After clone or large pulls, run `devenv shell -- roam init` or `roam index` so analysis stays accurate (`roam --help` for options).

Workflow:

1. First time in the repo: `roam understand` then `roam tour` (optional but useful).
2. Find a symbol: `roam search <pattern>`
3. Before changing a symbol: `roam preflight <name>` (blast radius, tests, fitness)
4. Files to read: `roam context <name>` (prioritized ranges)
5. Debugging a failure: `roam diagnose <name>`
6. After edits: `roam diff`

Also: `roam health`, `roam impact <name>`, `roam pr-risk`, `roam file <path>`.

Run `roam --help` for all commands. Use `roam --json <cmd>` for structured output.

---
> Source: [Industrial/id_effect](https://github.com/Industrial/id_effect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->

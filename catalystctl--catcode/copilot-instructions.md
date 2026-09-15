## catcode

> This file is injected into the standing prompt. Follow it.

# Catalyst Code — agent contract

This file is injected into the standing prompt. Follow it.

## Understand before you change
- Do not open files hoping. `knowledge` (context/search/symbol/related) → `lsp` definition/references → ranged `read_file`.
- Reuse the existing pattern. A second convention beside the existing one is a defect.
- Before changing an exported symbol, find callers.

## Change the smallest correct thing
- Prefer `ast_edit` for structural rewrites; `edit` for exact text.
- Native tools over bash. `git_*` for status/diff/log/show.
- One Rust core owns the agent loop (`core/src`). TUI is Go. Web is a thin hub. Do not add a second loop.
- No speculative abstractions, unused config, or error handling for impossible cases. If 200 lines could be 50, rewrite it.
- Don't "improve" adjacent comments, formatting, or dead code. Mention unrelated dead code; don't delete it unless asked. Every changed line should trace to the request.
- If a simpler approach exists, say so and prefer it.

## Finish and verify
- Complete in-scope work. No stubs or "as a next step" for work you can do now.
- Verify with the project's real check (`cargo test` in `core/`, targeted Go/web tests when those trees change).
- Cite file:line. Ground claims in what you read or ran.

## Multi-agent
- Unknown multi-area work: scout first, then planner, then worker. Reviewer after writers.
- Children escalate with `contact_supervisor`; do not ask the user mid-mission in goal/CEO mode.

---
> Source: [catalystctl/catcode](https://github.com/catalystctl/catcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-15 -->

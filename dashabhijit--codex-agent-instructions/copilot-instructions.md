## codex-agent-instructions

> This repository stores reusable `AGENTS.md` files for different Codex use

# AGENTS.md

## Purpose

This repository stores reusable `AGENTS.md` files for different Codex use
cases.

## Maintenance Instructions

- Keep reusable variants organized under `agents/<category>/<use-case>/AGENTS.md`.
- Keep `agents/` limited to files that end users can directly adopt in their
  own repositories.
- Put maintainers' guidance, rollout notes, and showcase material under `docs/`
  or root-level documentation instead of mixing it into `agents/`.
- Keep generic instruction files reusable across repositories.
- Move repo-specific guidance into dedicated variants instead of mixing it into
  generic files.
- Prefer concise, practical rules over long design philosophy sections.
- Keep examples short and easy to adapt.
- Update `agents/README.md` when adding or renaming instruction files.
- Do not overwrite an existing variant unless the change is intentional and
  reviewed.

## Editing Rules

- Preserve the intent of existing instruction files unless the task explicitly
  changes that intent.
- When creating a new variant, copy the closest existing variant and customize
  it for the target use case.
- Keep each variant immediately usable as a standalone `AGENTS.md` file.
- Keep naming consistent so files remain easy to discover.

---
> Source: [DashAbhijit/codex-agent-instructions](https://github.com/DashAbhijit/codex-agent-instructions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->

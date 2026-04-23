## sailing-passage-map

> User-global skill library path for Cursor agents


# Repo skill library

Committed repo skills live in `.github/skills` when that directory exists.
Authoring-oriented or older checkouts may still keep the editable source under
`.agents/skills`. `pnpm run skills:link` repairs the tracked agent-facing
symlinks so every tool resolves to the committed repo skill surface, preferring
`.github/skills` and falling back to `.agents/skills` only when needed.

- `.cursor/skills` → `../.github/skills` (preferred)
- `.codex/skills` → `../.github/skills` (preferred)
- `.agent/skills` → `../.github/skills` (preferred)
- `.claude/skills` → `../.github/skills` (preferred)
- legacy fallback: `../.agents/skills`

When work matches packaged guidance, read the relevant `SKILL.md` under
`.github/skills/<skill-id>/` when present, otherwise the legacy
`.agents/skills/<skill-id>/` tree.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/narduk-enterprises) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

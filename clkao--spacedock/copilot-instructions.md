## spacedock

> Spacedock uses its own workflow (`docs/plans/`) to manage development. This creates a distinction between project source code and workflow artifacts:

# Spacedock Development — Agent Guidelines

## Self-hosted workflow boundaries

Spacedock uses its own workflow (`docs/plans/`) to manage development. This creates a distinction between project source code and workflow artifacts:

**Ensigns may edit (project source code):**
- `skills/` — commission skill and other plugin skills
- `agents/` — plugin-shipped agent entry points
- `references/` — shared core and runtime adapters
- `scripts/` — test harnesses, utilities
- `mods/` — plugin-shipped mod templates
- `plugin.json` — plugin source

**Ensigns must NOT edit (workflow artifacts):**
- `docs/plans/README.md` — workflow schema, owned by refit
- `docs/plans/*.md` frontmatter — entity state, owned by first officer

**Why:** In a normal project using Spacedock, this distinction is obvious — ensigns edit project code, not workflow config. In Spacedock's own repo, the project code *is* workflow infrastructure, so the line needs to be explicit.

---
> Source: [clkao/spacedock](https://github.com/clkao/spacedock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->

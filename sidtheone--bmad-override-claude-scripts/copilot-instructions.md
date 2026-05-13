## bmad-override-claude-scripts

> Override scripts and hotfixes for the [BMAD Method](https://github.com/bmadcode/BMAD-METHOD) Claude Code integration. These scripts enhance the default BMAD pipeline with fixes, new review steps, and interaction enforcement.

# BMAD Override Claude Scripts

Override scripts and hotfixes for the [BMAD Method](https://github.com/bmadcode/BMAD-METHOD) Claude Code integration. These scripts enhance the default BMAD pipeline with fixes, new review steps, and interaction enforcement.

## Project Structure

```
.
├── 6.0.3/                          # Scripts for BMAD 6.0.3
├── 6.0.4/                          # Scripts for BMAD 6.0.4
├── 6.2.0/                          # Scripts for BMAD 6.2.0 (.claude/skills/ architecture)
├── compatibility.json              # Version registry (maps BMAD versions to scripts)
├── install-enhanced-sprint.sh      # Version-aware installer (local + remote)
├── enhanced-automated-sprint.md    # Root-level fallback (latest version)
├── claude-hotfix-interaction-style.md  # AskUserQuestion enforcement hotfix
└── README.md
```

Each version folder contains the same two scripts tailored to that BMAD release:
- `enhanced-automated-sprint.md` — Full sprint pipeline skill (11-step story lifecycle, or 12 with `--e2e`)
- `claude-hotfix-interaction-style.md` — Hotfix to enforce `AskUserQuestion` tool usage

**Note:** BMAD 6.2+ uses `.claude/skills/` (with `SKILL.md` entry points) instead of `.claude/commands/`. The installer auto-detects which architecture to target.

## Key Concepts

- **Version-aware installation**: The installer reads `_bmad/_config/manifest.yaml` from the target project to auto-detect the BMAD version, then pulls the matching scripts from the correct version folder.
- **Fallback chain**: Exact version match > closest lower version > `latest` from `compatibility.json` > root-level files.
- **Enhanced Automated Sprint**: A Claude Code skill (`/enhanced-automated-sprint`) that automates the full dev lifecycle: story creation, refinement, validation, TDD tests (unit + optional E2E), implementation, consolidated code review (BMAD 6.2 runs adversarial, edge case, and acceptance reviews internally), fixes, traceability, and sprint status updates.

## Adding a New BMAD Version

1. Create folder: `mkdir X.Y.Z`
2. Copy scripts: `cp <previous_version>/*.md X.Y.Z/`
3. Make version-specific changes in the new folder
4. Update `compatibility.json` — add version entry and set `"latest"`
5. Update root-level fallback files to match latest

## Conventions

- Root-level `.md` scripts are always kept in sync with the latest version folder
- `compatibility.json` is the single source of truth for version mapping
- The installer supports `--global`, `--force`, `--uninstall`, `--version=X.Y.Z`, and `--list`
- Scripts use Claude Code frontmatter format (`---` delimited YAML with `name` and `description`)
- The sprint pipeline uses Claude's Agent tool with model hints (Opus for planning/review, Sonnet for implementation)

## Repository

- GitHub: `sidtheone/BMAD-OVERRIDE-CLAUDE-SCRIPTS`
- License: MIT
- Branch: `main`

---
> Source: [sidtheone/BMAD-OVERRIDE-CLAUDE-SCRIPTS](https://github.com/sidtheone/BMAD-OVERRIDE-CLAUDE-SCRIPTS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->

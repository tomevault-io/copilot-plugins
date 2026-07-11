## rmm-skills

> A set of Claude Skills (structured instruction sets) that teach Claude to produce production-ready RMM scripts for Windows (PowerShell 5.1), macOS (zsh), and Linux (bash). The target RMM platforms are NinjaOne and Action1. Licensed MIT — designed for easy community adoption.

# RMM Skills for Claude — Project Guide

## What This Project Is

A set of Claude Skills (structured instruction sets) that teach Claude to produce production-ready RMM scripts for Windows (PowerShell 5.1), macOS (zsh), and Linux (bash). The target RMM platforms are NinjaOne and Action1. Licensed MIT — designed for easy community adoption.

## Project Structure

```
rmm-skills/
├── CLAUDE.md              # This file — project guide for Claude
├── README.md              # Public-facing docs and installation guide
├── CHANGELOG.md           # Version history (Keep a Changelog format)
├── LICENSE                # MIT
├── .gitignore
├── .github/
│   └── pull_request_template.md  # PR checklist for contributors
├── release.sh             # Build release zips (runs sanity checks first)
├── scripts/
│   └── check-release.sh   # Pre-release sanity check (version + conventions sync)
├── rmm-powershell/
│   ├── SKILL.md                       # PowerShell 5.1 skill (Windows)
│   ├── RMM-CONVENTIONS.md             # Shared conventions (copied into each skill folder)
│   └── NINJAONE-WYSIWYG-REFERENCE.md  # NinjaOne WYSIWYG field HTML/CSS reference
├── rmm-macos/
│   ├── SKILL.md                       # zsh skill (macOS Catalina 10.15+)
│   ├── RMM-CONVENTIONS.md
│   └── NINJAONE-WYSIWYG-REFERENCE.md
└── rmm-linux/
    ├── SKILL.md                       # bash 4.x+ skill (Linux)
    ├── RMM-CONVENTIONS.md
    └── NINJAONE-WYSIWYG-REFERENCE.md
```

Each platform folder is self-contained and distributed as a zip for Claude Desktop, or copied into `~/.claude/skills/` for Claude Code.

`RMM-CONVENTIONS.md` is **identical** across all three folders — it covers non-interactive execution, security, idempotency, logging, exit codes, input validation, code review mode, and response structure. **`rmm-powershell/RMM-CONVENTIONS.md` is the canonical copy** — edit that one first, then copy to the other two folders.

`NINJAONE-WYSIWYG-REFERENCE.md` is also **identical** across all three folders. Keep all three copies in sync when updating it.

## Key Conventions

### YAML Frontmatter (SKILL.md files)
- Version numbers **must be quoted**: `version: "1.2.0"` — unquoted YAML treats `1.2.0` as a float and truncates to `1.2`.
- `tags` is **not supported** in Claude Desktop skill metadata. Only `author` and `version` under `metadata:`.
- Keep frontmatter fields to: `name`, `description`, `metadata.author`, `metadata.version`.

### Skill Content
- Each SKILL.md starts with a persona ("You are a specialised, senior-level...") and scope guards ("When This Skill Applies / Does NOT Apply").
- Platform-specific templates, NinjaOne custom field references, Action1 attribute references, cross-platform translation notes, and good/bad examples are all inline in the SKILL.md.
- Keep the language British English for consistency with the existing content (e.g., "specialised", "defence-in-depth", "organisation").

### Version Bumping
- All three SKILL.md files share the same version number. Bump all three together.

## Release Workflow

Every release ships **all three zips** regardless of which skills changed — users can grab any single zip from any release and have the latest.

1. Bump version in all three `SKILL.md` frontmatter blocks.
2. Add a new entry to `CHANGELOG.md`.
3. Run `bash release.sh` to build zips — this runs sanity checks automatically, then packages `rmm-powershell.zip`, `rmm-macos.zip`, `rmm-linux.zip`.
5. Zips are gitignored — rebuild from source each release.
6. Create a GitHub release with `gh release create vX.Y.Z *.zip` and attach the zips.

## Known Quirks

- **NinjaOne `$0` / `SCRIPT_NAME` bug:** NinjaOne copies scripts to temp paths before execution, so `basename "$0"` or `${0:t}` resolves to a meaningless name. All macOS/Linux templates hardcode `SCRIPT_NAME` instead.
- **Action1 + `[CmdletBinding()]`:** Action1 does not support `[CmdletBinding()]` or `param()` blocks. The PowerShell skill has a separate Action1 template.
- **NinjaOne custom fields require SYSTEM/root:** Custom field access (read and write) does not work in user context on any platform.
- **Skill name vs directory mismatch:** The `name` field in SKILL.md frontmatter (e.g., `rmm-powershell-scripts`) doesn't match the directory name (`rmm-powershell`) or zip filename (`rmm-powershell.zip`). This works fine currently — Claude doesn't enforce a match — but may need fixing if the SKILL.md manifest becomes stricter in future.
- **`ninjarmm-cli` not in PATH on Linux:** Bare `ninjarmm-cli` won't resolve. Use `./ninjarmm-cli`, the full path `/opt/NinjaRMMAgent/programdata/ninjarmm-cli`, or `"$NINJA_DATA_PATH/ninjarmm-cli"`.

## Development Workflow

Changes to skills are driven by real-world usage feedback: use the skill in Claude, then ask Claude how the skill performed and what could use improvement or clarification. This feedback loop identifies gaps in the instructions that wouldn't be obvious from reading the files alone.

## Commit Messages

Keep it simple — clear, descriptive messages with no strict format required.

## Editing Guidelines

- This is a documentation-only project — no executable code in the repo itself. The skills instruct Claude how to *generate* code.
- When adding support for a new RMM platform, update: the relevant SKILL.md(s), RMM-CONVENTIONS.md (all three copies), and README.md.
- When adding a new platform skill (e.g., `rmm-freebsd/`), follow the existing folder structure: `SKILL.md` + `RMM-CONVENTIONS.md` copy.
- Keep examples practical and representative of real RMM deployment scenarios.

## Platform Expansion

- **New RMM platforms:** The maintainer doesn't have access to other RMM platforms, but PRs adding platform-specific quirks are welcome. Current coverage: NinjaOne (full), Action1 (basic).
- **New OS targets:** Currently limited to what NinjaOne and Action1 support (Windows, Linux, macOS). New OS skills would only be added if an RMM platform adds scripting support for that OS (e.g., ChromeOS).

---
> Source: [DeusMaximus/rmm-skills](https://github.com/DeusMaximus/rmm-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->

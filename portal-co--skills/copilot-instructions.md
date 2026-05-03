## skills

> This repository is a shared registry of [Agent Skills](https://agentskills.io/specification). Each subdirectory containing a `SKILL.md` file is an installable skill.

# Portal Solutions Agent Skills Registry

This repository is a shared registry of [Agent Skills](https://agentskills.io/specification). Each subdirectory containing a `SKILL.md` file is an installable skill.

## What You Can Do Here

- **Discover skills**: Browse subdirectories or check `registry.json` for the full index
- **Load a skill**: Read the `SKILL.md` in the relevant subdirectory — it contains full instructions
- **Invoke a skill**: Use `/skill:<name>` if your harness supports slash commands
- **Add a skill**: Follow `CONTRIBUTING.md` — create a directory, add `SKILL.md`, run `scripts/sync-registry.sh`

## Skill Index

See `registry.json` for a machine-readable list of all skills with names and descriptions.

## Structure

```
skills/
├── README.md               # Human-readable docs
├── AGENTS.md               # This file — agent project context
├── CONTRIBUTING.md         # How to add a new skill
├── import-config.json      # Bulk import configuration (excludes, include_only, source_dirs)
├── imports.json            # Manifest of imported skills (source SHAs, sync dates)
├── registry.json           # Machine-readable skill index (auto-generated)
├── .github/
│   └── workflows/
│       └── sync-skills.yml # Automated weekly import + PR workflow
├── scripts/
│   ├── install.sh          # Symlinks skills into ~/.agents/skills/
│   ├── import-skills.sh    # Imports skills from portal-co repos
│   └── sync-registry.sh    # Regenerates registry.json from SKILL.md frontmatter
├── <skill-name>/           # Locally authored skill
│   └── SKILL.md
└── <repo-name>/            # Skills imported from portal-co/<repo-name>
    └── <skill-name>/
        └── SKILL.md
```

## Installation

Run `./scripts/install.sh` to symlink this registry into your agent harness skills directory.

Or add this directory to your harness settings manually:
```json
{ "skills": ["/path/to/portal-hot/skills"] }
```

---
> Source: [portal-co/skills](https://github.com/portal-co/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->

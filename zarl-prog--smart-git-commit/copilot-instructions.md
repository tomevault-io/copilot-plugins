## smart-git-commit

> smart-git-commit/       # kebab-case directory name

# Smart Git Commit — Agent Skills Manifest

## Skill Directory Structure

```
skills/
  smart-git-commit/       # kebab-case directory name
    SKILL.md              # Required: skill definition and phases
    scripts/              # Executable bash scripts
    references/           # Supporting docs, loaded on demand
    templates/            # Drop-in config files for projects
    tests/                # Test fixtures and scenarios
```

## Naming Conventions

- **Skill directories**: kebab-case
- **SKILL.md**: always uppercase, always this exact filename
- **Scripts**: kebab-case.sh (bash) or kebab-case.mjs (Node)
- **References**: kebab-case.md
- **Templates**: dotfiles and `.example` suffixes (`.gitmessage`, `CLAUDE.md.example`)

## Script Requirements

- **Shebang**: `#!/usr/bin/env bash`
- **Error handling**: `set -euo pipefail` for strict mode
- **Progress/logs** → stderr
- **Machine-readable results** → stdout as JSON
- **Cleanup traps** for temp files
- **Exit 0** = success, **exit 1** = failure (blocks caller)
- **Color**: Use ANSI color codes (RED for errors, GREEN for success, YELLOW for warnings)

### Output JSON Schema

All scripts output JSON to stdout:

```json
{
  "status": "pass" | "fail" | "not_found",
  "findings": [
    {
      "file": "path/to/file.ext",
      "line": 42,
      "pattern": "matched pattern description"
    }
  ],
  "message": "Human-readable summary"
}
```

## SKILL.md Rules

- Under 500 lines
- YAML frontmatter with name + description (description must be "pushy")
- Progressive disclosure: SKILL.md is entry point, details in references/
- Phases numbered, each references the right script or reference file

## Progressive Disclosure Pattern

1. **Level 1 — Frontmatter** (always in context, ~100 words)
2. **Level 2 — SKILL.md body** (in context when skill triggers, <500 lines)
3. **Level 3 — references/** (loaded on demand, unlimited depth)
4. **Level 4 — scripts/** (executed, only output enters context)
5. **Level 5 — templates/** (drop-in configs, user copies and customizes)

---
> Source: [Zarl-prog/Smart-git-commit](https://github.com/Zarl-prog/Smart-git-commit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->

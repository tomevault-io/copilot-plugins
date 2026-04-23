## social-media-skills

> This document defines the conventions and requirements for skills in this repository.

# Agent Skills Specification

This document defines the conventions and requirements for skills in this repository.

## Repository Structure

```
social-media-skills/
├── .claude-plugin/
│   └── marketplace.json       # Plugin manifest
├── skills/
│   ├── social-media-context-sms/
│   │   └── SKILL.md
│   ├── content-strategy-sms/
│   │   └── SKILL.md
│   └── <skill-name>/
│       └── SKILL.md
├── tools/
│   └── REGISTRY.md            # Tool integrations registry
├── AGENTS.md                  # This file
├── CLAUDE.md                  # Agent guidelines
├── LICENSE
├── README.md
└── validate-skills.sh         # Validation script
```

Skills are grouped by layer in `skills/` but have no required subdirectory structure beyond `<skill-name>/SKILL.md`.

## Skill Requirements

### YAML Frontmatter

Every `SKILL.md` must begin with a YAML frontmatter block:

```yaml
---
name: skill-name
description: One to three sentence description of what this skill does and when to use it.
metadata:
  version: "1.0.0"
---
```

Field constraints:

| Field | Type | Constraints |
|---|---|---|
| `name` | string | 1–64 characters, kebab-case |
| `description` | string | 1–1024 characters |
| `metadata.version` | string | semver format (e.g. `"1.0.0"`) |

### Naming Conventions

- **Lowercase only** — no uppercase letters anywhere in skill names or directory names
- **Alphanumeric and hyphens only** — no spaces, underscores, dots, or special characters
- **Kebab-case** — words separated by single hyphens (e.g. `hook-writer-sms`, not `hookwriter` or `hook_writer`)
- The `name` field in frontmatter must match the directory name exactly

### File Size

- Each `SKILL.md` must be **under 500 lines**
- If a skill grows beyond this limit, split guidance into focused sub-sections or extract repeated patterns into shared references

### Content Structure

A well-formed `SKILL.md` should include:

1. YAML frontmatter (required)
2. A brief purpose statement
3. Inputs — what context or parameters the skill expects
4. Steps — numbered or structured instructions the agent follows
5. Outputs — what the skill produces
6. Examples (optional but encouraged)
7. See also — links to related skills

## Git Workflow

Use **conventional commits** for all changes to this repository:

| Prefix | When to use |
|---|---|
| `feat:` | Adding a new skill or significant new capability |
| `fix:` | Correcting errors in skill instructions or metadata |
| `update:` | Improving or expanding an existing skill |
| `docs:` | Changes to README, AGENTS.md, CLAUDE.md, or other docs |
| `chore:` | Tooling, config, or maintenance changes |

Example commit messages:
- `feat: add hook-writer-sms skill`
- `update: expand platform-strategy-sms with Bluesky guidance`
- `fix: correct frontmatter version field in post-writer-sms`
- `chore: update validate-skills.sh to check metadata.version`

## Validation

Run the validation script before committing:

```bash
./validate-skills.sh
```

The script checks:
- YAML frontmatter exists with `name` and `description`
- `name` is kebab-case and within 1–64 characters
- `description` is within 1–1024 characters
- File is under 500 lines

All checks must pass before merging to `main`.

---
> Source: [blacktwist/social-media-skills](https://github.com/blacktwist/social-media-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->

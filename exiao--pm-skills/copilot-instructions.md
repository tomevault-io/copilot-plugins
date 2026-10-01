## pm-skills

> A public repo of product/R&D skills for Claude Code, [Hermes Agent](https://github.com/NousResearch/hermes-agent), Codex, and other skill-aware agents. Every directory at root is a **stage** of the R&D process (except `.github/` and `scripts/`), and each stage contains related skills.

# pm-skills Repo Conventions

A public repo of product/R&D skills for Claude Code, [Hermes Agent](https://github.com/NousResearch/hermes-agent), Codex, and other skill-aware agents. Every directory at root is a **stage** of the R&D process (except `.github/` and `scripts/`), and each stage contains related skills.

## Repo Structure

```
stage-name/
├── DESCRIPTION.md            # Stage description (with name + description frontmatter)
└── skill-name/
    ├── SKILL.md              # Entry point (required)
    ├── references/           # Detailed docs, checklists, examples
    ├── scripts/              # Deterministic code (Python, JS, bash)
    └── assets/               # Templates, images, static resources
```

## Stages (the R&D pipeline)

| Stage | What's inside |
|-------|--------------|
| **strategy** | Diagnose the real obstacle and size up the field: strategy-kernel, competitive-analysis |
| **uxr** | Understand the user: synthetic-userstudies, journey-mapping |
| **pm** | Spec and justify: pitch-creator, architecture-diagram |
| **design** | Make it good: design-review, design-iteration |
| **qa** | Prove it works: verify-feature, dogfood, adversarial-ux-test |
| **launch** | Ship and price: launch-strategy, pricing-strategy |
| **thinking** | Cross-cutting reasoning used at every stage: quick-brainstorm, another-perspective |

## Skill Conventions

- `SKILL.md` must have YAML frontmatter with `name` and `description`.
- `description` is a routing instruction for the model, not a human summary. Include trigger phrases.
- Keep `SKILL.md` under 500 lines. Move details to `references/`.
- No README.md, CHANGELOG.md, or human-facing docs inside skill directories.

## Frontmatter Format

```yaml
---
name: my-skill
description: What this skill does and when to invoke it. Include trigger phrases.
---
```

Only `name` and `description` are required.

## Writing Good Descriptions

The `description` field is the primary routing signal. When a user says "help me with X," the agent picks a skill based on description match.

**Do:**
- Start with what the skill does, then when to use it
- Include literal trigger phrases ("Use when: map the flow, where do users drop off, storyboard this")
- Be specific about scope boundaries ("For visual execution, use design-iteration instead")

**Don't:**
- Write marketing copy ("A powerful toolkit for...")
- Be vague ("Helps with product tasks")
- Duplicate another skill's trigger phrases

## Rules

1. **No hardcoded credentials.** Use `$ENV_VAR_NAME` for tokens, API keys, product IDs. This repo is public.
2. **No personal or proprietary data.** No emails, phone numbers, real subscription/project IDs, revenue figures, internal URLs, or personal filesystem paths. These skills are derived from a private library and must read as generic and portable for any person or company.
3. **Update generated docs** when adding, removing, or renaming skills. Run `python scripts/generate_catalog.py`; it regenerates `CATALOG.md` and the README stage table.
4. **Prefer portable paths.** Use workspace-relative or well-known config paths. Avoid paths to personal directories.
5. **Don't rename product names.** "Hermes Agent", "Claude Code", "Codex" are real product names; use them as-is.

## CI

`claude-code-review.yml` runs on every PR. It checks frontmatter validity, hardcoded secrets or personal data, broken internal references, accuracy of CLI/tool references, and scope coherence. Fix real issues before requesting merge.

## PR Guidelines

- One logical change per PR. Branch from `main`. Don't stack branches.
- If CI flags issues, fix them before requesting merge.

---
> Source: [exiao/pm-skills](https://github.com/exiao/pm-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->

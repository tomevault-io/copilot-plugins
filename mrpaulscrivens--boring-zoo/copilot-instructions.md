## boring-zoo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Boring Zoo is a Claude Code skills ecosystem for building a business around your worldview. Every skill reads from a shared **World Code** foundation (7 elements stored in `world-code/*.md` files), ensuring all output is personalized to the user's voice, transformation, method, offers, positioning, content strategy, and conversion path.

Installed via: `npx skills add MrPaulScrivens/Boring-Zoo`

## Architecture

### World Code Framework (Foundation Layer)

The core architectural concept: users run `/world-code-start` to build 7 foundation files that every other skill depends on:

| Element | File | Purpose |
|---------|------|---------|
| Voice | `world-code/voice.md` | Tone, rhythm, hard rules |
| Climax | `world-code/climax.md` | Transformation promise, before/after states |
| Method | `world-code/method.md` | Unique methodology |
| Creation | `world-code/creation.md` | Offers, pricing, delivery |
| Crown | `world-code/crown.md` | Positioning declaration |
| Conversation | `world-code/conversation.md` | Content themes, wrong beliefs |
| Crossing | `world-code/crossing.md` | Conversion path, entry points |

`world-code-start/SKILL.md` is the orchestrator — it checks which elements exist and routes users to the next one.

### Skill Structure

Every skill is a single `SKILL.md` file inside its own directory with YAML frontmatter:

```
skill-name/
└── SKILL.md    # Frontmatter (name, description, version) + full instructions
```

Skills that produce content follow this pattern:
1. Load World Code files (check they exist, prompt user to run `/world-code-start` if missing)
2. Ask only task-specific questions (World Code pre-answers audience, voice, product, positioning)
3. Generate output calibrated to user's voice
4. Offer alternatives (3+ options for critical elements)

### References

`references/` contains 60+ shared markdown files used across skills:
- `boring-ref-*.md` — World Code-specific references (headline formulas, landing page blueprints, objection patterns)
- `ref-*.md` — Domain references (copywriting frameworks, SEO patterns, analytics, cold email)

Skills link to these with relative paths like `references/boring-ref-headline-formulas.md`.

### Plugin Configuration

- `.claude-plugin/plugin.json` — Defines all skills and their paths for CLI distribution
- `.claude-plugin/marketplace.json` — Marketplace publishing metadata

## Key Conventions

- **Skill naming**: `boring-*` for content/marketing skills, `world-*` for World Code foundation skills
- **All skill logic lives in SKILL.md** — no code files, no configs per skill
- **Directory names use lowercase kebab-case**
- **World Code is required context** — skills must check for it and refuse to produce generic output without it
- **Notable framework fusion**: `boring-email-sequence` maps Soap Opera Sequence (SOS) beats to World Code elements (Hook→Climax, Backstory→Voice+Crown, Wall→Conversation, Epiphany→Method, Resolution→Climax+Creation+Crossing)

## Adding a New Skill

1. Create `skill-name/SKILL.md` with YAML frontmatter (`name`, `description`, optional `metadata.version`)
2. Include a "Before Starting — Load Your World" section if the skill needs World Code
3. Add shared frameworks/patterns to `references/` rather than inlining them
4. Register the skill in `.claude-plugin/plugin.json`
5. Verify the skill works both with and without World Code present

## Acknowledgments

Skills structure and reference format started from [marketingskills](https://github.com/coreyhaines31/marketingskills) by Corey Haines.

---
> Source: [MrPaulScrivens/Boring-Zoo](https://github.com/MrPaulScrivens/Boring-Zoo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->

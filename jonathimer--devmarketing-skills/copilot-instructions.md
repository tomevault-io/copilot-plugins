## devmarketing-skills

> Guidelines for AI agents working in this repository.

# AGENTS.md

Guidelines for AI agents working in this repository.

## Repository Overview

This repository contains **Agent Skills** for AI agents following the [Agent Skills specification](https://agentskills.io/specification.md). Skills install to `.agents/skills/` (the cross-agent standard). These skills are specifically designed for **marketing to developers** — DevRel, developer tools, APIs, SDKs, and technical products.

- **Name**: Developer Marketing Skills
- **Focus**: Marketing to developers (DevRel, dev tools, APIs, technical products)
- **License**: MIT

## Repository Structure

```
devmarketing-skills/
├── skills/                # Agent Skills
│   └── skill-name/
│       ├── SKILL.md       # Required skill file
│       ├── README.md      # Quick overview
│       └── references/    # Optional supporting docs
├── AGENTS.md              # This file
├── CONTRIBUTING.md
├── LICENSE
└── README.md
```

## Build / Lint / Test Commands

**Skills** are content-only (no build step). Verify manually:
- YAML frontmatter is valid
- `name` field matches directory name exactly
- `name` is 1-64 chars, lowercase alphanumeric and hyphens only
- `description` is 1-1024 characters

## Agent Skills Specification

Skills follow the [Agent Skills spec](https://agentskills.io/specification.md).

### Required Frontmatter

```yaml
---
name: skill-name
description: What this skill does and when to use it. Include trigger phrases.
metadata:
  version: 1.0.0
---
```

### Frontmatter Field Constraints

| Field         | Required | Constraints                                                      |
|---------------|----------|------------------------------------------------------------------|
| `name`        | Yes      | 1-64 chars, lowercase `a-z`, numbers, hyphens. Must match dir.   |
| `description` | Yes      | 1-1024 chars. Describe what it does and when to use it.          |
| `license`     | No       | License name (default: MIT)                                      |
| `metadata`    | No       | Key-value pairs (author, version, etc.)                          |

### Name Field Rules

- Lowercase letters, numbers, and hyphens only
- Cannot start or end with hyphen
- No consecutive hyphens (`--`)
- Must match parent directory name exactly

**Valid**: `developer-onboarding`, `hacker-news-strategy`, `x-devs`
**Invalid**: `Developer-Onboarding`, `-onboarding`, `developer--onboarding`

### Skill Directory Structure

```
skills/skill-name/
├── SKILL.md        # Required - main instructions (<500 lines)
├── README.md       # Recommended - quick overview and usage
├── references/     # Optional - detailed docs, templates, examples
├── scripts/        # Optional - executable code
└── assets/         # Optional - templates, data files
```

## Foundation Skill

The `developer-audience-context` skill is the foundation. All other skills reference it first to understand:
- Developer persona (role, seniority, tech stack)
- Where they hang out (Reddit, HN, Discord, Twitter)
- Pain points and problems
- Current alternatives
- Voice and tone

When creating new skills, include a "Before You Start" section that checks for `.agents/developer-audience-context.md`.

## Writing Style Guidelines

### Structure

- Keep `SKILL.md` under 500 lines (move details to `references/`)
- Use H2 (`##`) for main sections, H3 (`###`) for subsections
- Use bullet points and numbered lists liberally
- Short paragraphs (2-4 sentences max)
- Include tables for reference data and comparisons

### Tone

- Direct and instructional
- Second person ("Check if the developer-audience-context exists")
- Technical but accessible — assume readers understand developer concepts
- No marketing fluff — developers see through it

### Developer Marketing Principles

When writing skills, remember that developers:
- Skip landing pages → Go straight to docs, GitHub, API reference
- Distrust marketing speak → Want technical accuracy and honest tradeoffs
- Value transparency → Open source, public roadmaps, honest pricing
- Research thoroughly → Read HN comments, Reddit threads, GitHub issues
- Influence purchasing → Even when not the buyer, they're the decision-maker

### Formatting

- Bold (`**text**`) for key terms
- Code blocks for examples, commands, and templates
- Tables for reference data
- No excessive emojis

### Clarity Principles

- Clarity over cleverness
- Specific over vague
- Active voice over passive
- One idea per section

### Description Field Best Practices

The `description` is critical for skill discovery. Include:
1. What the skill does
2. When to use it (trigger phrases)
3. Related skills for scope boundaries

```yaml
description: When the user wants to write technical tutorials or quickstart guides. Use when the user mentions "tutorial," "quickstart," "getting started guide," or "code walkthrough." For general DevRel content, see devrel-content.
```

## Tool Integrations

Skills reference various tools relevant to developer marketing:

| Category | Tools |
|----------|-------|
| **Listening & Monitoring** | Octolens |
| **Documentation** | Mintlify, ReadMe, GitBook |
| **Analytics** | Plausible, PostHog, Amplitude |
| **Community** | Discord, Slack |
| **Email** | Resend, Loops, Customer.io |

## Git Workflow

### Branch Naming

- New skills: `feature/skill-name`
- Improvements: `fix/skill-name-description`
- Documentation: `docs/description`

### Commit Messages

Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:

- `feat: add hacker-news-strategy skill`
- `fix: improve clarity in developer-onboarding`
- `docs: update README with new skill`

### Pull Request Checklist

- [ ] `name` matches directory name exactly
- [ ] `name` follows naming rules (lowercase, hyphens, no `--`)
- [ ] `description` is 1-1024 chars with trigger phrases
- [ ] `SKILL.md` is under 500 lines
- [ ] README.md included with quick overview
- [ ] References `developer-audience-context` where appropriate
- [ ] No sensitive data or credentials

## Skill Categories

Skills are organized into these categories:

| Category | Focus |
|----------|-------|
| **Foundation** | Developer audience context (referenced by all others) |
| **Content & Community** | DevRel content, tutorials, newsletters, community, advocacy |
| **Distribution & Discovery** | HN, Reddit, GitHub, Twitter/X, LinkedIn, YouTube |
| **Developer Experience** | Docs, SDKs, API onboarding, sandboxes, changelogs |
| **Growth & Acquisition** | SEO, directories, lead gen, hackathons, ads |
| **Competitive Intelligence** | Listening, competitor tracking, alternatives pages |
| **Conversion & Activation** | Signup, onboarding, free tier, pricing |
| **Lifecycle & Retention** | Email sequences, churn, power users |

When adding new skills, follow the naming patterns of existing skills in that category.

## Cross-References

Skills should reference related skills in their "Related Skills" section. Common patterns:

- `devrel-content` ↔ `technical-tutorials` ↔ `developer-seo`
- `developer-listening` ↔ `competitor-tracking` ↔ `alternatives-pages`
- `developer-onboarding` ↔ `docs-as-marketing` ↔ `api-onboarding`
- `developer-signup-flow` ↔ `developer-onboarding` ↔ `free-tier-strategy`

## Adding New Skills

1. Create directory: `skills/skill-name/`
2. Create `SKILL.md` with valid YAML frontmatter
3. Create `README.md` with quick overview
4. Add to appropriate category in main `README.md`
5. Update ASCII diagram if skill changes category structure
6. Submit PR following the checklist above

---
> Source: [jonathimer/devmarketing-skills](https://github.com/jonathimer/devmarketing-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->

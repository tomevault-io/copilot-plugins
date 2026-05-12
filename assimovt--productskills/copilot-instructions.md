## productskills

> this skill. This is always loaded in context; body only loads when triggered.

# ProductSkills

Open-source AI agent skills for product teams. 16 skills built on real frameworks (Mom Test, Shape Up, Obviously Awesome, Teresa Torres, Linear Method, YC).

## What This Repo Is

A collection of SKILL.md files that teach AI agents (Claude Code, Cursor, Codex, Devin) how to do product management work — discovery, strategy, prioritization, scoping, and PRD writing. Every skill is 50-150 lines, opinionated, and framework-driven.

## Architecture

```
skills/[skill-name]/SKILL.md          # The skill (50-150 lines, YAML frontmatter)
skills/[skill-name]/references/*.md   # Optional deep-dive templates (loaded on demand)
```

Skills use Anthropic's SKILL.md standard:
- YAML frontmatter with `name` and `description` only
- Body: framework + ALWAYS/NEVER/CRITICAL rules
- No role-setting ("You are an expert..."), no output format sections

## Skill Design Rules

1. **50-150 lines per SKILL.md.** Context window is shared with user's code. No bloat.
2. **Opinionated.** ALWAYS/NEVER/CRITICAL rules. "DO X" not "consider doing X."
3. **Framework-driven.** Every skill is grounded in a real methodology (Teresa Torres, Shape Up, Mom Test, April Dunford, etc.). Never make up frameworks.
4. **Startup-calibrated.** Assumes teams of 3-50. No SAFe, no RACI, no velocity theater.
5. **Examples > explanations.** One concrete example beats three paragraphs of theory.
6. **No fluff files.** No README, CHANGELOG, or docs per skill folder.

## SKILL.md Format

```yaml
---
name: skill-name
description: >
  One paragraph (~50 words). Must contain trigger phrases for when to use
  this skill. This is always loaded in context; body only loads when triggered.
---
```

Body pattern (adapt per skill):
```
[Opening line: what + philosophy, 1-2 sentences]

## [Core Framework]
[The methodology. This is 60%+ of the skill.]

## Guidelines
[ALWAYS/NEVER/CRITICAL rules]
```

## Quality Bar

Anthropic's `frontend-design` skill is the benchmark: ~60 lines, immediately actionable, uses NEVER/CRITICAL for guardrails, philosophy over procedures. Trust the agent to execute within guidelines.

## The 16 Skills

### Discovery & Research
- `user-interview` — Mom Test + YC Five Questions
- `problem-validation` — Frequency x Intensity x WTP scoring
- `jtbd-analysis` — Jobs-to-be-done, Forces of Progress
- `research-synthesis` — Atomic research: nuggets to insights
- `opportunity-mapping` — Teresa Torres OST

### Strategy & Positioning
- `competitor-analysis` — Feature matrix, positioning gaps
- `product-positioning` — April Dunford's Obviously Awesome
- `strategy-doc` — Playing to Win + Strategy Kernel

### Prioritization & Scoping
- `feature-prioritization` — RICE + Linear enablers/blockers
- `scope-cutting` — Shape Up appetite + scope hammering
- `bet-sizing` — Shape Up pitch format + Type 1/2 decisions

### The PRD
- `prd-writing` — Evidence-first, 800-1200 words, P0/P1/P2

### Launch & Measure
- `launch-plan` — Launch tiers with checklists
- `metrics-framework` — North Star + counter-metrics
- `experiment-design` — Hypothesis-driven A/B tests
- `roadmap-planning` — Now/Next/Later, outcome-based

## Commands

```bash
# Test a skill in Claude Code
npx skills add . --skill prd-writing

# Lint markdown
npx markdownlint-cli2 "skills/**/*.md"
```

## Important

- NEVER add implementation details to skills (specs, code snippets). Skills cover product thinking BEFORE the spec.
- NEVER use corporate PM language (SAFe, velocity, story points, RACI, stakeholder alignment).
- NEVER add emojis to skill files.
- ALWAYS cite the framework source in the skill (e.g., "Shape Up by Basecamp", "The Mom Test by Rob Fitzpatrick").
- ALWAYS test skills with real prompts before shipping.

---
> Source: [assimovt/productskills](https://github.com/assimovt/productskills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->

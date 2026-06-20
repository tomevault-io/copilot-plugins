## clay-claude-code-skill-pack

> This repo contains four Claude Code skills for working with Clay, the GTM data

# CLAUDE.md — clay-claude-code-skill-pack

This repo contains four Claude Code skills for working with Clay, the GTM data
enrichment and outbound sequencing platform. Each skill is a `.md` file in
`.claude/skills/`. Load the relevant skill before any Clay-related task.

## Skills in This Repo

| Skill | When to use it |
|-------|---------------|
| `clay-table-architect` | Designing a new Clay table from scratch |
| `clay-enrichment-waterfall` | Building or auditing enrichment column sequences |
| `clay-ai-column-prompts` | Writing or improving AI column prompts |
| `clay-outbound-export` | Preparing a table for export to a sequencing tool |

## How to Load a Skill

Say: "Use the clay-table-architect skill" or "Read .claude/skills/clay-table-architect.md"
before describing your task. Claude will read the skill file and apply it.

## Who This Is For

VP Sales, founders, RevOps leads, and GTM engineers at B2B SaaS and tech companies
who use Clay to build prospecting lists, enrich accounts, and feed outbound sequences.

## What These Skills Are Not

These are not Clay documentation. They are opinionated instruction sets for Claude Code
that encode best practices, decision logic, and quality checks so Claude produces
output that actually works in production Clay tables — not theoretical scaffolds.

## Testing

Run tests from the `tests/` folder to verify each skill produces valid output.
See `tests/README.md` for instructions.

---
> Source: [forma-norden/clay-claude-code-skill-pack](https://github.com/forma-norden/clay-claude-code-skill-pack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->

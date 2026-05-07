## myclaude-creator-engine

> This repository contains the **MyClaude Studio Engine** — an open-source creation pipeline for Claude Code products.

# AGENTS.md

This repository contains the **MyClaude Studio Engine** — an open-source creation pipeline for Claude Code products.

## What This Does

Guides users through a complete pipeline: research a domain, create a product structure, fill it with expertise, validate quality against 20 structural patterns, test in sandbox, package, and publish to a marketplace where anyone can install with one command.

## Key Entry Points

| Path | Purpose |
|------|---------|
| `CLAUDE.md` | Engine identity, boot sequence, 15 skills, rules |
| `.claude/skills/` | 15 built-in slash commands (/onboard, /scout, /create, /fill, etc.) |
| `templates/` | Product templates for all 13 types |
| `product-dna/` | Structural DNA patterns per product type |
| `references/` | Specifications, quality gates, best practices, pipeline docs |
| `workspace/` | Active product builds (gitignored) |
| `llms.txt` | Machine-readable project summary |
| `.claude-plugin/marketplace.json` | Plugin manifest for marketplace discovery |
| `.well-known/ai-plugin.json` | AI agent discovery metadata |

## How to Use

```bash
myclaude studio                    # Install via CLI
# or
git clone https://github.com/myclaude-sh/myclaude-creator-engine
cd myclaude-creator-engine && claude
```

Then run `/onboard` inside Claude Code. The engine activates from `CLAUDE.md`.

## Architecture (one sentence)

A state machine that guides users through onboard, scout, create, fill, validate, test, package, publish — with quality gates at each transition and marketplace distribution at the end.

## 13 Product Types

skill, agent, squad, workflow, minds, system, claude-md, hooks, statusline, output-style, design-system, application, bundle

## When NOT to Use This

- User just wants to **install** an existing tool → use `myclaude install <slug>` directly
- User needs general Claude Code configuration help → not what this does
- User wants a one-off prompt → just type it, no engine needed
- User wants pre-built configs → this is a factory, not a warehouse

## Links

- Marketplace: https://myclaude.sh
- CLI: `npm i -g @myclaude-cli/cli`
- License: MIT

---
> Source: [myclaude-sh/myclaude-creator-engine](https://github.com/myclaude-sh/myclaude-creator-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->

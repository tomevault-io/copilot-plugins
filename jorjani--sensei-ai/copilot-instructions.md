## sensei-ai

> AI-powered personal coaching tool. Uses Claude Code to gather data about your work and coach you through structured reflection.

# AI Coach

AI-powered personal coaching tool. Uses Claude Code to gather data about your work and coach you through structured reflection.

## How It Works

1. `config.yaml` defines your data sources and chosen coach
2. Source adapters (`sources/`) tell you how to gather data (git, TODOs, priorities)
3. Coaching personas (`coaches/`) define the review structure and voice
4. Commands (`.claude/commands/`) orchestrate the flow

## Commands

- `/review` -- Run a weekly review with your configured coach
- `/review last` -- Review the previous week
- `/setup` -- Interactive first-run configuration
- `/add-coach [name]` -- Generate a coaching persona (e.g., `/add-coach Naval Ravikant`)

## First Run

If no `config.yaml` exists, run `/setup` to create one. Or copy `config.example.yaml` to `config.yaml` and edit manually.

## Project Layout

```
coaches/          Coaching personas (Mochary, Grove, or run /add-coach)
sources/          Data source adapters (git, TODOs, priorities, calendar, GitHub, journal)
scripts/          Automation (scheduled reviews)
results/          Review outputs (gitignored)
config.yaml       Your personal config (gitignored)
```

## Extending

- **Add a source:** Drop a markdown file in `sources/` describing what to gather and how. Reference its config keys.
- **Add a coach:** Drop a markdown file in `coaches/` defining the voice, structure, and output format.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jorjani)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jorjani)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

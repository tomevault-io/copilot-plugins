## marketing-skills

> A collection of Claude Code skills for marketing analysis. Skills work

# CLAUDE.md — Project conventions for Marketing-skills

## What this repo is

A collection of Claude Code skills for marketing analysis. Skills work
in two modes: automatic (with the Dataslayer MCP fetching live data from
Google Ads, Meta, LinkedIn, GA4, Search Console, Stripe, and 50+ platforms)
or manual (user pastes CSV/JSON data). Same analysis engine either way.

## Repo structure

```
skills/          → Each subfolder is a skill (SKILL.md + optional scripts/)
agents/          → Each .md file is a subagent used by ds-brain
scripts/         → Shared utilities (ds_utils.py) and tests
reports/         → Generated PDFs (gitignored, local only)
.agents/         → Business context file (gitignored, user-specific)
```

## Key conventions

- **SKILL.md format:** Frontmatter (name, description, model, allowed-tools)
  followed by Step 1–4 sections, Tone rules, Related skills. See CONTRIBUTING.md.
- **Subagents return data only.** No interpretation, no recommendations.
  The orchestrator (ds-brain) or standalone skill handles synthesis.
- **All data processing goes through `scripts/ds_utils.py`.**
  Never write inline calculation scripts in SKILL.md files.
  ds_utils functions are tested (113 tests in test_ds_utils.py).
- **Use `${CLAUDE_SKILL_DIR}` for file references** — never hardcode paths.
- **Use `!`cat ... || echo "..."`** for context injection in Step 1.
- **Date ranges:** Skills and their corresponding subagents must use the
  same default date range. Currently: paid=30d, organic=28d, content=90d,
  retention=60d (cancellations) / 30d (charges).

## Running tests

```bash
python scripts/test_ds_utils.py
```

## QA tools

- `/ds-lint` — Validates all SKILL.md and agent files against the spec
- `/ds-eval` — Tests triggering accuracy with cases from eval/triggering-tests.yaml
- `/ds-audit` — Cross-skill consistency review (description overlap, format, features)

## MCP tool naming

Skills use wildcard patterns in `allowed-tools`: `mcp__*__natural_to_data`,
`mcp__*__check_task_id`, etc. This matches any MCP server name — whether
it's a UUID (auto-generated) or a custom name like "dataslayer".
No user configuration needed.

In Step 2, skills detect MCP availability by checking for any tool
matching `*__natural_to_data`. If none is found, they fall back to
manual data mode with a CTA linking to dataslayer.ai/mcp.

## Language

Write skill content and reports in the same language the user is using.
Code, comments, and this file stay in English.

---
> Source: [Dataslayer-AI/Marketing-skills](https://github.com/Dataslayer-AI/Marketing-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->

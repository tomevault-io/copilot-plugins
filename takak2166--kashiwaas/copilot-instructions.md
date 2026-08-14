## kashiwaas

> <!-- Do not restructure or delete sections. Update individual values in-place when they change. -->

# Agent Guidelines

<!-- Do not restructure or delete sections. Update individual values in-place when they change. -->

## Core Principles

- **Do NOT maintain backward compatibility or fallback** unless explicitly requested. Break things boldly.
- **Line budget:** Non-blank, non-HTML-comment lines = **instruction body** (target **30–50**). **Whole file** ≤**75** lines (includes blanks, fences, HTML notes). Offload depth to `docs/`.
- **Plan-first:** Non-trivial work (multi-file, behavior/config/bot/hooks/CI changes): write a plan first; **do not edit files** until the user approves. **Plan optional** only for trivial fixes (typos, comments, obvious nits).

---

## Project Overview

**Project type:** Slack message ingestion, Elasticsearch storage, daily/weekly reports and Kibana visuals, plus a Bolt bot for `@kashiwaas` (Cursor Cloud Agents API)
**Primary language:** Python 3.12+ (see `pyproject.toml`)
**Key dependencies:** Poetry; slack-sdk / slack-bolt; elasticsearch; requests — full list in `pyproject.toml`

---

## Commands

```bash
# Development
poetry install && poetry run python -m src.cli   # app entry (local)
docker-compose up -d                                 # full stack

# Testing / lint
make test && make lint                               # pytest; ruff

# Docs
README.md (setup & ops), docs/README.md (design index)
```

---

## Code Conventions

- Follow the existing patterns in the codebase
- Prefer explicit over clever
- Delete dead code immediately

---

## Architecture

```
src/cli/             CLI entry (``python -m src.cli``) and fetch helpers
src/slack/           Slack client and messages
src/es_client/       Elasticsearch
src/analysis/        Daily/weekly analysis and visualization
src/bot/             Report posting, alerts, KashiwaaS bot
src/cursor/          Cursor Cloud Agents API client
src/kibana/          Kibana integration
scripts/             Index setup, Kibana import, etc.
```

---

## Maintenance Notes

<!-- This section is permanent. Do not delete. -->

**Keep this file lean and current:**

1. **Remove placeholder sections** (sections still containing `[To be determined]` or `[Add your ... here]`) once you fill them in
2. **Review regularly** - stale instructions poison the agent's context
3. **CRITICAL: Respect the line budget** (see Core Principles) — move detailed docs to separate files and reference them
4. **Update commands immediately** when workflows change
5. **Rewrite Architecture section** when major architectural changes occur
6. **Delete anything the agent can infer** from your code

**Remember:** Coding agents learn from your actual code. Only document what's truly non-obvious or critically important.

---
> Source: [takak2166/KashiwaaS](https://github.com/takak2166/KashiwaaS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->

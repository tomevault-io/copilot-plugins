## crm-dedupe-plugin

> This repository packages a standalone CRM dedupe plugin, and it is designed to work cleanly in Claude Code.

# CRM Dedupe Plugin — Claude Code Guide

This repository packages a standalone CRM dedupe plugin, and it is designed to work cleanly in Claude Code.

The repo includes both the plugin/command layer and the dedupe engine files:

- `agents/`
- `config/`
- `db/`
- `pipeline/`
- `review/`
- `scheduler/`
- `tests/`

## How To Work Here

- Read this file first, then the relevant file under `skills/*/SKILL.md`.
- Use the `.claude/commands/` slash commands when available.
- Run CRM Dedupe commands from this repo root unless the user explicitly points to another engine with `CRM_DEDUPE_AGENT_REPO`.
- Keep this plugin repo free of secrets, CRM exports, databases, customer records, and private GTM notes.
- Preserve the original CRM Dedupe Agent scoring contract. The plugin is an orchestration layer, not a replacement scoring engine.
- Keep commands configurable with `CRM_DEDUPE_AGENT_REPO` or `--repo-root`; do not hard-code local machine paths.
- Default to dry-run instructions and require explicit caps before live merge commands.
- Use `$merge-safety-review` before any live CRM write.

## Claude Code Slash Commands

Plugin commands live in `commands/` at the plugin root (loaded by Claude Code via `.claude-plugin/plugin.json`):

- `/crm-dedupe-orchestrator`
- `/contact-dedupe-agent`
- `/company-dedupe-agent`
- `/hubspot-dedupe-backfill`
- `/merge-safety-review`
- `/daily-crm-hygiene`
- `/verify-scoring-contract`

These commands mirror the Codex skills in `skills/`.

## Validation

Run this before publishing changes:

```bash
python3 scripts/verify_original_scoring_contract.py
python3 -m py_compile scripts/company_fallback_summary.py scripts/verify_original_scoring_contract.py
python3 -m pytest tests/ -q
```

Expected scoring verifier output:

```text
OK: original scoring contract preserved
```

## Public Packaging Checklist

- No `.env` files.
- No CSV/TSV exports.
- No database files.
- No local absolute paths.
- No private customer examples.
- `README.md` remains useful for both Codex and Claude Code users.

---
> Source: [Andytoizer/crm-dedupe-plugin](https://github.com/Andytoizer/crm-dedupe-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->

## pdf-agent

> uv run pytest {Path to the specific unit test} -v

# AGENTS.md

## Setup
uv sync

## Unit Tests
uv run pytest {Path to the specific unit test} -v

## Integration Tests
uv run pytest {Path to the specific integration test} -v

## Lint
uv run ruff check . --fix && uv run ruff format .

## Rules
- Do NOT edit migrations
- Do NOT change terraform files
- Never commit secrets
- Prefer minimal diffs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/huangrao121)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/huangrao121)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

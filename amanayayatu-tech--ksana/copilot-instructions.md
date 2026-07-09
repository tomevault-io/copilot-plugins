## ksana

> This repository is `ksana`, a local multi-agent investment decision system.

# AGENTS.md

## Project Context

This repository is `ksana`, a local multi-agent investment decision system.
It has a deterministic Python pipeline around `research-agent`, three partner
agents, `chairman`, `red-team`, `orchestrator`, and the local FastAPI/Jinja Web
UI in `webui.py`.

## Current Module Map

- `business_agents/research_agent/`: price-signal scan, pull-request prompts,
  and cold-start historical research planning.
- `business_agents/trading_agent.py`: shared partner-agent execution path.
- `chairman/opportunity/`: Opportunity Screener scoring, status routing,
  non-consensus thesis checks, and configurable score caps.
- `chairman/core/brief_assembler.py`: assembles Opportunity Memo payloads from
  validated partner recommendations and screener output.
- `red_team/core/risk_policy.py`: configurable risk-budget policy with fatal
  flaw always forcing zero budget.
- `orchestrator/core/learning.py`: decision cases, score snapshots, outcome
  snapshots, stock timelines, and monthly learning reviews.
- `orchestrator/reporting/html_renderer.py`: report-center HTML rendering.
- `templates/`: local web workspace pages.

## Product Boundary

- The system is research-only and human-in-the-loop.
- `trial_candidate` and `conviction_candidate` mean human review / paper
  tracking / human-approved tracking-position candidates, not buy signals.
- Opportunity Score is a heuristic triage and later evaluation input, not an
  investment recommendation.
- Red Team risk budget is a boundary description; it never places orders.

## Codex CLI Provider Rules

- When invoked by the backend as an LLM provider, return only the requested final
  content. If the caller asks for JSON, return strict JSON with no Markdown
  fences or commentary.
- Do not bypass the Python validators or output schemas. Business outputs must
  still pass the existing Pydantic/schema validation in the repository.
- Do not directly write business artifacts into `data/`, including historical
  briefs, recommendations, run logs, or Perplexity result files. The Python
  pipeline is responsible for writing those files after validation.
- Do not call Perplexity, web search, data APIs, or broker/trading APIs. The
  Perplexity workflow is human-in-the-loop: output prompts, wait for Nepha's
  manual result, and consume local `data/perplexity_results/` files only.
- Do not make trading decisions for Nepha. Summarize, validate, flag risk, and
  preserve the final decision boundary.
- If evidence is missing or unverified, keep the conservative watch/abstain
  posture required by the methodology and deployment layer.
- Preserve `legacy_verdict` and existing report keys unless a compatibility
  migration and tests are included.

## Local Commands

- Use `uv run ...` for repository commands when shell execution is needed.
- Prefer `--no-llm` smoke tests for deterministic local verification.
- Keep generated caches and transient logs separate from source changes.
- Before pushing documentation or product-surface changes, run:
  `uv run ruff check` and `uv run pytest -q`.

---
> Source: [amanayayatu-tech/ksana](https://github.com/amanayayatu-tech/ksana) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->

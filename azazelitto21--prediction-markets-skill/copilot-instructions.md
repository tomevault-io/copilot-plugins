## prediction-markets-skill

> Guidance for AI agents working in or with this repository.

# AGENTS.md

Guidance for AI agents working in or with this repository.

## What this is

A self-contained, read-only skill giving agents accurate access to Kalshi and Polymarket
public market data: discovery, orderbooks, price history, implied probability, cross-venue
matching. The skill itself lives in `skills/prediction-markets/` (SKILL.md + references +
scripts). Install instructions for end-user agents: see `llms-install.md`.

## If you're an agent USING the skill

Read `skills/prediction-markets/SKILL.md` — it has the routing table, workflows and output
contract. Key rules:

- Run the bundled scripts; do NOT write Kalshi/Polymarket API calls from memory. Endpoints
  drift and models hallucinate them — that is the exact problem this repo exists to solve.
- Every script prints one JSON envelope to stdout; errors are one JSON line on stderr with
  exit codes 2/3/4/5/6/7 (usage/network/rate-limit/geo/not-found/schema).
- When numbers look wrong, read `skills/prediction-markets/references/gotchas.md` first.

## If you're an agent CONTRIBUTING here

- Rules live in `CLAUDE.md` (hard constraints: stdlib-only, read-only, endpoints only from
  `docs-raw/`).
- `docs-raw/` holds snapshots of the official API docs and is the ONLY endpoint ground truth.
  Regenerate it from live docs before touching API code; never trust memorized endpoints.
- Definition of done per script: runs against the live API, valid JSON in <5s, tested on 3+
  distinct queries, listed in SKILL.md's routing table.
- Evals live in `evals/` (trigger-eval.json, tasks.json). Run them before rewriting SKILL.md.

## Scope guarantee

This repo is read-only by design. Trading logic is out of scope permanently (a separate
skill may exist for it); do not add order placement, auth flows, or key handling here.

---
> Source: [azazelitto21/prediction-markets-skill](https://github.com/azazelitto21/prediction-markets-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->

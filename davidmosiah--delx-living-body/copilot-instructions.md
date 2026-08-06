## delx-living-body

> Quick reference for AI agents working with this MCP server.

# AGENTS.md — delx-living-body

Quick reference for AI agents working with this MCP server.

## What it is

A **meta-MCP server**: it composes other locally-installed Delx Wellness connectors (WHOOP, Oura, Garmin, etc.) into a single unified surface. Pure data composition — no LLM calls inside the server.

## Tools (6)

1. `living_body_status` — which connectors are detected? Safe, no subprocess spawning.
2. `living_body_ask` — main tool. Spawns children in parallel, returns synthesized answer. **Requires `explicit_user_intent: true`**.
3. `living_body_daily_brief` — markdown brief from each connector's daily summary.
4. `living_body_compose_context` — normalized `delx-wellness-context/v1` shape merged across sources.
5. `living_body_health_check` — all 15 known connectors with install hints for missing ones.
6. `living_body_capabilities` — self-description and per-connector availability matrix.

## Recommended agent flow

1. Call `living_body_capabilities` once at session start.
2. Call `living_body_status` to see what's installed.
3. For a specific question → `living_body_ask` (always with `explicit_user_intent: true`).
4. For raw merged context to hand off to another tool → `living_body_compose_context`.
5. For a daily-stand-up-style summary → `living_body_daily_brief`.
6. If sources are missing → show the user the `install_hint` from `living_body_health_check`.

## Rules

- **Always pass `explicit_user_intent: true`** to `living_body_ask`. The tool spawns multiple subprocesses and should never be called speculatively.
- **Default privacy mode is `structured`.** Only use `raw` when the user explicitly asks for vendor-native payloads.
- **Treat output as operational context, not medical advice.**
- Don't ask the user to install every connector. Only suggest installing what's needed for the current question.
- The synthesizer is rule-based — its reasoning trace has stable `rule_id`s (`rec_low`, `bb_high`, etc.). When you summarize for the user, you can reference these.

## Confidence interpretation

- `low` — 0 or 1 sources, or sources disagree by >35 points on overlapping scores.
- `medium` — 2 non-conflicting sources.
- `high` — 3+ non-conflicting sources.

## Troubleshooting

- "no sources used" → run `living_body_health_check`. Probably nothing is installed or every child timed out.
- A child timed out → its package may be slow to npx-install. Pre-install it: `npx -y <package> setup`.
- Cache feels stale → `DELX_LIVING_BODY_NO_CACHE=true` or delete `~/.delx-living-body/cache.sqlite`.

---
> Source: [davidmosiah/delx-living-body](https://github.com/davidmosiah/delx-living-body) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->

## unreal-ai-dev

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this directory is

A **research workspace** for evaluating how to use Claude (Claude Code + Claude API) to develop Unreal Engine 5 projects. It is **not** a UE project itself — there is no `.uproject`, no `Source/`, no engine source. Do not assume UE tooling (UBT, RunUAT, Live Coding) is set up here unless the user says otherwise.

There are no build, lint, or test commands.

## Primary artifact

`REPORT.md` at the repo root — 341‑line, fully‑cited report covering:
1. Inventory of MCP servers for UE (chongdashu, flopperam, kvick, Natfii, NAJEMWEHBE, prajwalshettydev, runreal, GenOrca, soft-ue-cli, ue-llm-toolkit, gdep, etc.) with star counts, UE version support, install patterns
2. Fab/Marketplace AI plugins (Ludus, CLAUDIUS, Rekall, LocoAI, Aura, NWIRO, Muddy Terrain Gen AI, AI Chat Plus)
3. Practitioner workflows (Rider + Claude Code, NVIDIA AST chunking, StraySpark setup guides)
4. Top OSS repos categorized
5. Honest reviews and pitfalls (Blueprint binary problem, file-by-file reading, context blowup, MCP drops under load)
6. Concrete phased onboarding recommendation for this user

Before re-researching any of these topics, **read REPORT.md first** — it was produced May 2026 with live WebSearch/WebFetch evidence.

`.omc/research/research-20260511-unreal-ai-claude/` — the original sciomc session: a copy of the report, `state.json` metadata, and runtime artifacts. Treat as an append-only archive of how the report was generated. Future research sessions should follow the same `research-YYYYMMDD-<slug>/` pattern there.

## Recommended stack (from report §6)

If the user pivots to actually scaffolding a UE project in this repo:
- **UE 5.6** (stable plugin coverage; skip 5.7 until plugin ecosystem catches up)
- **One MCP server**: `runreal/unreal-mcp` (lowest setup, uses UE Python Remote Execution) **or** `chongdashu/unreal-mcp` (most popular, 1.9k★, marked EXPERIMENTAL)
- **`gdep` as a second MCP** so Claude queries a codebase map instead of reading files alphabetically
- A project `CLAUDE.md` ≤150 lines covering project structure, naming, current sprint, confirmation rules
- A `.claudeignore` excluding `Binaries/ Intermediate/ Saved/ DerivedDataCache/ *.uasset *.umap`
- **Model**: Opus 4.6 for new-feature first prompts, Sonnet 4.6 for daily grind

## Known pitfalls to call out proactively

- Blueprints are binary `.uasset` — Claude **cannot** read them via plain file tools; require an MCP that parses them (Natfii/UnrealClaude, ue-llm-toolkit, any §1 MCP server) or `gdep` for structural maps
- Claude Code reading UE Source file-by-file blows up tokens; always `.claudeignore` `Engine/` and similar
- MCP connections drop under heavy batch ops — modify assets in zone batches, not 5000 at once
- Editor must be running **before** `claude` starts, or MCP can't connect
- UE 5.6 / 5.7 has API drift the models still occasionally hallucinate around (especially MovieScene); pin the engine version in CLAUDE.md

## OMC state

`.omc/state/` is managed by oh-my-claudecode — do not hand-edit. `.omc/research/` is the canonical place for research session artifacts; future research sessions should follow the same `research-YYYYMMDD-<slug>/` pattern.

---
> Source: [getarobo/unreal-ai-dev](https://github.com/getarobo/unreal-ai-dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-24 -->

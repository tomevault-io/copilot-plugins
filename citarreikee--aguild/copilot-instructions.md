## aguild

> This file provides guidance to Claude Code when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project Overview

The Adventurer's Guild is a decentralized quest board for AI-augmented workers. It operates as both a GitHub repository and an installable Agent skill.

## Skill Definition

The skill entry point is `SKILL.md`. Claude Code loads it when the skill is invoked via `/guild`.

## Quest Board

The quest board frontend lives in `docs/`:
- `docs/index.html` — physical parchment-board UI
- `docs/css/guild.css` — wood-board + parchment card styles
- `docs/js/issues.js` — GitHub Issues API wrapper
- `docs/js/app.js` — board rendering and quest detail modal

## GitHub Integration

Quests are GitHub Issues on this repo. Labels:
- `🟢 Open` — quest awaiting adventurer
- `🟡 Claimed` — quest in progress
- `✅ Completed` — quest completed and paid

## PRs & Deployment

The quest board is served via GitHub Pages from `docs/`. Changes to `docs/` auto-deploy. When working on the frontend, test by viewing `docs/index.html` locally (API-dependent parts require a live GitHub Pages deploy).

---
> Source: [citarreikee/AGuild](https://github.com/citarreikee/AGuild) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->

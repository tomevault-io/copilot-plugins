## toto-huat

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Toto Huat** — An app that analyzes past Singapore Toto lottery results to find potential bias in draw generation and identify higher-probability number picks.

Singapore Toto: Players pick 6 numbers from 1–49. Each draw produces 6 winning numbers + 1 additional number.

## Architecture

Decoupled frontend and backend — the UI/UX can be built independently from the data/analysis layer.

### Frontend
- **Framework:** React with TypeScript
- **UI Components:** ShadCn (built on Radix UI + Tailwind CSS)
- **Build Tool:** Vite
- **Styling:** Tailwind CSS v4

### Backend
- **Runtime:** Node.js with TypeScript
- **Framework:** Express or Hono
- **Data:** Historical Toto draw results (scraped or API-sourced)
- **Analysis:** Statistical analysis of draw patterns, frequency analysis, bias detection

### Monorepo Structure (planned)
```
apps/
  frontend/     # React + ShadCn UI
  backend/      # API server + analysis engine
packages/
  shared/       # Shared types, constants, utilities
```

## Key Domain Concepts

- **Draw Result:** 6 winning numbers (1–49) + 1 additional number, with draw date and prize info
- **Frequency Analysis:** How often each number appears across historical draws
- **Hot/Cold Numbers:** Numbers drawn more/less frequently than expected
- **Overdue Numbers:** Numbers that haven't appeared in recent draws
- **Pattern Analysis:** Consecutive numbers, sum ranges, odd/even distribution, number group distribution
- **Bias Detection:** Statistical tests (chi-squared, etc.) to identify non-uniform distribution in draws

---
> Source: [jumbomochi/toto-huat](https://github.com/jumbomochi/toto-huat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
